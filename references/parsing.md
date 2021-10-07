# Overview of parsing
Recommended reading: [Rules](../references/rules.md)

**Warning:** Difficult and janky stuff ahead.

**Warning 2:** If you are looking to make a mod that involves parsing in any way, I suggest you proceed with caution. Parsing is very involved with a lot of systems built in, some of which might feel hacky. If your mod introduces a new behavior that affects how parsing works (e.g., words with custom syntax/text type, words that change parsing direction/location, etc), expect that you have to deal with these various systems. 

Again, **this guide doesn't cover everything about parsing.** It only gives an overview on the basic steps involved in parsing. The best source of figuring out specific details is the lua files themselves.



## Step 1: When parsing happens
Parsing happens whenever a function called `code()` is called. Located in `rules.lua`, `code()` is the starting point on where all parsing happens. The [Baba wiki](https://babaiswiki.fandom.com/wiki/Advanced_rulebook) has a really good page on when `code()` is called (Under "Order of Operations").

Whenever `code()` is called, the game checks if a global variable, called `updatecode`, is set to 1. If so, the game clears all tables that store rule related data and starts parsing. Otherwise, the game skips parsing and leaves the rule tables unchanged. `updatecode` is set to 1 whenever the game detects a change in the state of the level that *might* affect the set of rules being formed (e.g., a text object is moved/teleported/destroyed/created).



## Step 2: Firstwords
After checking `updatecode=1`, the game builds a list called `firstwords`. A *firstword* is a text unit in the level that the parser determines as a possible candidate to start a sentence. *This does not account for syntax of the text unit,* meaning that `facing` and `push` could be a firstword even though it isn't valid syntax.

### Initial firstwords
At the start of `code()`, the game determines what I call the "initial firstwords". It scans through all text units in the level and selects some to be initial firstwords. There is a simple criteria for this (based on the function `codecheck()`):

1. There is a text unit ahead of it (1 tile right or down of it)
2. There is *not* text unit behind it (1 tile left or up of it)
---
- Side note: if you analyze where it determines the initial firstwords in `code()`, the game can actually add a text unit *twice* into `firstwords`. This is possible if the text unit is an initial firstword both horizontally and vertically.
---

The reason of my distinction of "initial firstwords" is because not all firstwords are initial firstwords. This will be explained in Step 4, Part B.

### docode()
After putting all initial firstwords into a list called `firstwords`, it passes that list into `docode()`. While `code()` is the entry point to start parsing, `docode()` is the meat of the parsing logic. The next few sections will be within the scope of `docode()`. A brief overview on what the function does.
1) Use the list of initial firstwords to determine all strings of text units (including all combinations from stacked texts).
2) Go through each string to extract valid rules.
3) Construct rule objects, as defined [here](rules.md#rule-format), and submit to `featureindex` to make the rule avaliable globally.



## Step 3: `calculatesentences()`
On submitting initial firstwords, the first main function in `docode()` that gets called with the input is `calculatesentences()`. 

In basic terms, the game's code refers a "sentence" as a list of texts, *regardless of syntax*. The main output of `calculatesentences()` consists of a list of sentences, where each "sentence" is a table like the example below:
```lua
-- This corresponds to a string of text units laid out in this order "keke push baba is you blue".
{
    -- Each entry is a "word", formatted like {text name, text type, {unitid of text unit}, width}
    {"keke", 0, {<unitid>}, 1},
    {"push", 2, {<unitid>}, 1},
    {"baba", 0, {<unitid>}, 1},
    {"is",   1, {<unitid>}, 1},
    {"you",  2, {<unitid>}, 1},
    {"blue", 2, {<unitid>}, 1},
}
```
- Notice how the full sentence has some texts before and after the "baba is you" that are not valid in syntax. This is expected. `calculatesentences()` does not care about syntax. It only returns lists of horizontal or vertical adjacent text units.
- The "width" parameter (the last entry of each word object), is always 1 for normal texts. If the `text name` is actually spelled out with letter texts, it is the number of the letter texts involved.

### Implementation of `calculatesentences()`
You can divide `calculatesentences()` into two phases
#### Phase 1

- Starting from a firstword, iterate within the level in the indicated parsing direction (either right or down) until there are no more text units.
- For each space iterated, get a list of all text units in that space, and add it to a bigger list: `sents`.
  - After phase 1, `sents` would look like this:
  
  ```lua
  {
      -- {{unitid}, width, wordname, wordtype, unitdir}
      { {{<unitid>}, 1, "bird", 0, 2},     {{<unitid>}, 1, "baba", 0, 2}  },
      { {{<unitid>}, 1, "is",   1, 2} },
      { {{<unitid>}, 1, "word", 2, 2} },
  }
  -- This corresponds with "(bird&baba) is word", where bird and baba are stacked. Parsing direction is down (indicated by unitdir = 2)
  ```
- Record information needed for Phase 2.

#### Phase 2
From `sents`, extract all combinations of texts. With the above example, this will turn into "bird is word" and "baba is word". The output list of sentences would look like this:

```lua
{
    -- sentence 1
    {
        -- {text name, text type, {unitid of text unit}, width}
        {"bird", 0, {<unitid>}, 1},
        {"is",   1, {<unitid>}, 1},
        {"word", 2, {<unitid>}, 1},
    },
    -- sentence 2
    {
        -- {text name, text type, {unitid of text unit}, width}
        {"baba", 0, {<unitid>}, 1},
        {"is",   1, {<unitid>}, 1},
        {"word", 2, {<unitid>}, 1},
    },
}
```

## Step 4: Syntax Checking and Reinsertion into `firstwords`

### Part A
The output of `calculatesentences()` gets passed into this step. For each sentence, the game scans through each word in order while inserting those that follow a valid syntax into a final sentence. This "final sentence" is a list that gets submitted to Step 5 for insertion into `featureindex`. The below diagram illustrates how the game updates its internal state in order to check syntax when scanning through a sentence. Some extra notes:
- Valid sentences always start from Node 0 to Node -1 without getting stuck.
- If syntax checking gets stuck in a node other than -1, end the syntax checking by going to node -1 and go to Part B of Step 4.

<img src="../images/Baba syntax diagram.png" width="1920"/>

---
### Part B
Consider this string of text blocks:
```
lonely keke is facing baba is you
```
You can make `calculatesentences()` output this "sentence" simply by moving the text blocks in game to form this arrangement. However, notice that the only valid sentence, "baba is you", starts in the middle of the sentence. If you run this sentence through the syntax checking in Part A, it will abruptly stop on "facing" since it is invalid syntax (Node 3 if you are looking at the diagram). It's tempting to think that if we hit an invalid state in Part A, then just throw the sentence away. But then we wouldn't be able to parse the "baba is you" in the middle of the sentence.

Part B is meant to handle cases where there are valid sub sentences within an invalid sentence. This is done by inserting a word from the current sentence *back into* `firstwords`. It also saves the current sentence parsed from `calculatesentences()`, so that it doesn't have to run Step 3 again. This effectively "restarts" Part A from a new point within the current sentence, allowing "baba is you" to be extracted from the above example. Recall in Step 2 that I differentiated the "Initial firstwords" from other firstwords. The "other firstwords" come from this reinsertion process.

There are a few cases that determine *which* text gets reinserted into `firstwords`. I'll add the relevant code below just for reference. But I won't actually use it to explain all cases. Instead, I'll use some examples that cover some cases. I suggest you go through each example and try to match it with the relevant logic within the snippet.

The word in `[]` will denote the current word that stopped the syntax checking in Part A. The word in `()` will denote which word gets reinserted into `firstwords`.
- `[push] (me) is move` - Since `push` is the first word, restart at the next word, `me` to get `me is move`.
- `baba is (keke) [is] you` - Parse `baba is keke` then stop at `is`. Since `is` is a verb and `keke` is a noun, restart parsing at `keke` to get `keke is you`
- `bird is (water) [on] baba is sink` - Parse `bird is water` then stop at `on`. Since `on` is infix and `water` is a noun, restart parsing at `water` to get `water on baba is sink`
- `ghost is (not) not [lonely] baba is pull` - We found a `not` while parsing up to `lonely`. Restart parsing at the first `not` we encountered to get `not not lonely baba is pull`.
- `ice is (lava) not [near] water is orange` - Parse `ice is lava` then stop at `near`. The last "safe" word (a word that isn't `not`) is `lava`. So restart parsing at `lava` to get `lava not near water is orange`
- `baba is ([is]) you` - Restart parsing on the same `is` since there is no other special case that handles this. But ultimately, you'll get no sentences from this.

<details>
<summary>Relevant code block. Click to expand</summary>

```lua
--[[ 
    - wordid = index of current word, 
    - existing_wordid = index of firstword within sentence, 
    - firstrealword = if we reached a Node that's greater than 0
    - #sent = number of words in sentence
    - nexts[2] = tiletype of the next word
    - nexts[3] = unitid of the next word
    - notids = stores first encountered "not" text unitid. Gets cleared when either: 
        - encountering a valid non-noun word
        - encountering two consecutive nouns 
    - notslot = the index of the "not" text stored in notids (remember that lua indexes start at 1)

 ]]
if (wordid < #sent) then
    if (wordid > existing_wordid) then
        if (#notids > 0) and firstrealword and (notslot > 1) and ((tiletype ~= 7) or ((tiletype == 7) and (prevtiletype == 0))) and ((tiletype ~= 1) or ((tiletype == 1) and (prevtiletype == 0))) then
            -- MF_alert(tostring(notslot) .. ", not -> A, " .. unique_id .. ", " .. sent_id)
            local subsent_id = string.sub(sent_id, (notslot - existing_wordid)+1)
            table.insert(firstwords, {notids, dir, notwidth, "not", 4, sent, notslot, subsent_id})
            
            if (nexts[2] ~= nil) and ((nexts[2] == 0) or (nexts[2] == 3) or (nexts[2] == 4)) and (tiletype ~= 3) then
                -- MF_alert(tostring(wordid) .. ", " .. tilename .. " -> B, " .. unique_id .. ", " .. sent_id)
                subsent_id = string.sub(sent_id, j)
                table.insert(firstwords, {s[3], dir, tilewidth, tilename, tiletype, sent, wordid, subsent_id})
            end
        else
            if (prevtiletype == 0) and ((tiletype == 1) or (tiletype == 7)) then
                -- MF_alert(tostring(wordid-1) .. ", " .. sent[wordid - 1][1] .. " -> C, " .. unique_id .. ", " .. sent_id)
                local subsent_id = string.sub(sent_id, wordid - existing_wordid)
                table.insert(firstwords, {sent[wordid - 1][3], dir, tilewidth, tilename, tiletype, sent, wordid-1, subsent_id})
            elseif (prevsafewordtype == 0) and (prevsafewordid > 0) and (prevtiletype == 4) and (tiletype ~= 1) and (tiletype ~= 2) then
                -- MF_alert(tostring(prevsafewordid) .. ", " .. sent[prevsafewordid][1] .. " -> D, " .. unique_id .. ", " .. sent_id)
                local subsent_id = string.sub(sent_id, (prevsafewordid - existing_wordid)+1)
                table.insert(firstwords, {sent[prevsafewordid][3], dir, tilewidth, tilename, tiletype, sent, prevsafewordid, subsent_id})
            else
                -- MF_alert(tostring(wordid) .. ", " .. tilename .. " -> E, " .. unique_id .. ", " .. sent_id)
                local subsent_id = string.sub(sent_id, j)
                table.insert(firstwords, {s[3], dir, tilewidth, tilename, tiletype, sent, wordid, subsent_id})
            end
        end
        
        break
    elseif (wordid == existing_wordid) then
        if (nexts[3][1] ~= -1) then
            -- MF_alert(tostring(wordid+1) .. ", " .. nexts[1] .. " -> F, " .. unique_id .. ", " .. sent_id)
            local subsent_id = string.sub(sent_id, j+1)
            table.insert(firstwords, {nexts[3], dir, nexts[4], nexts[1], nexts[2], sent, wordid+1, subsent_id})
        end
        
        break
    end
end
```
</details>

## Step 5: Submitting rules into `featureindex`

At this point, most of the rules that will be submitted into `featureindex` are finalized[1]. The game will then convert each rule into the format specified [here](rules.md#rule-format). The actual details of how this is done I'll leave out. But at a high level, say we have `baba facing keke is you`, it will determine which text is the "target" (baba), the verb (is), and the effect (you), as well as the conditions (facing keke). Finally, the game submits this formatted rule by calling `addoption()` (located in `rules.lua`), which does the necessary steps for inserting into `featureindex`.

---
- [1] There is a small section of code that goes through the list of finalized rules and removes duplicate rules. "Duplicate" in the sense that the set of text unitids that make up the sentence are the same.
---

## Step 6: Other parsing
Here is where `docode()` ends. By this point all rules extracted directly from looking at the in-game text blocks have been added to `featureindex`. However, there are a few more functions that add more rules to featureindex to support special words. 

### subrules()
Handles "all" and "mimic". Both of these are common in that if a rule involving them is added to `featureindex`, it causes more rules to be added. So if "baba is all", then it adds "baba is keke", "baba is rock", "baba is water" ...etc
### grouprules()
Adds rules based on "group" dynamics. This function is complex in of itself so I will not bother covering it.
### postrules()
This does a variety of things. A minor function is determine if the rule sound effect should play. But it also handles rules that can *modify* existing rules in `featureindex`. A few examples.
- `baba is baba` modifies all rules like `baba is rock` in order to cancel them. It does this by adding a special condition called `never` which always evaluates to false.
- `keke is not move` does the same method to cancel any `keke is move` rules.


## Final comments
As you can tell by the 6 steps, there's *alot* involved. And it barely scratches the surface. There is so much more to the parsing process that I left out for the sake of making this overview. If you decide to dive deeper into the source code, prepare to be confused and overwhelmed as heck. I sure was confused when trying to research on making omni text *(\*shivers)*.