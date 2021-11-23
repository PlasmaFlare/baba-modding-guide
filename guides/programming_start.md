# Basics of using Lua to mod Baba Is You
Before continuing, I recommend reading [setting up your enviornment for Baba modding](setup.md).


Programming anything to modify the game starts with making a Lua file. You can make as many Lua files you want to implement your mod. I recommend making them within a levelpack in order to self-contain the mod's effects within the levelpack. Create each Lua file in `<your levelpack folder>/Lua` and name it whatever you want (as long as it ends with `.lua`). The game will load your Lua files in alphanumeric order.

## Modifying the game
There are two main ways to mod the game with Lua. (You can still create your own functions/variables, but the following methods directly modify the game's behavior):

### Method 1: Override Functions from the source code
You can override existing functions from the game's Lua files to get the most flexibility in functionality. The downside is having to resolve code conflicts, either from updating Baba or from merging two mods. 

Most of these functions are contained in Lua files stored at `<Baba install dir>/Data`. The names of these files are pretty simple, such as `movement.lua` and `rules.lua`. Click [here](../references/lua%20files.md) for a brief overview of most of the main Lua files.

Say you want to override the function `testcond()` found in `conditions.lua` (One common reason to do this is to implement your own conditional words, like "on", "near" and "lonely"). Copy the entire function and paste it in your own Lua file you made in `<your levelpack folder>/Lua`. You can then modify your copy as you wish and the game will use your version instead of the one defined in `conditions.lua`, but *only* whenever you load the levelpack in game. You can do a similar process for other functions defined in the main game Lua files. Just know that you don't have to copy the entire lua file; only the functions you want to override.

You can copy multiple overwritten functions from different main game files into a single Lua file. The game doesn't care about which Lua file you place your overwritten functions. All it does is go through each Lua file in `<your levelpack folder>/Lua` and any overwritten functions you added will replace the ones defined by the main game.

### Method 2: Mod hooks
To try to reduce the amount of conflicts between two different mods, Hempuli added a small mod hook system. This works by inserting your custom function into different lists of functions. These lists of functions will then be called by the game, in the order the functions were inserted, at certain points in the game's code. The downside of modhooks is their limited range and restricted access to local variables in the game's code. This makes mod hooks alright for code that can be compatable with other mods, but they are also restrictive in which areas of the game you can affect.

Here's an example of adding a custom function to a mod hook that executes when the level starts (taken directly from `<baba install dir>\Data\Worlds\debug\Lua\tutorial.lua`)

```lua
table.insert(mod_hook_functions["level_start"],
	function()
		timedmessage("Starting a new level!")
	end
)
```
The full list of modhooks is defined in `modsupport.lua`. In the above code, replace `"level_start"` with whichever modhook you want to register your own function.

#### Finding when a modhook is called
If you want to see where the modhooks are actually being called, search for the text:
```lua
do_mod_hook("<replace with your modhook>"
```
Note that some modhooks will be called by MMF directly instead of in a Lua file. So you might not see the above search for some of the modhooks.

### Should you use mod hooks or override functions?
Personally, I try to gravitate towards using mod hooks and/or my own functions and variables when implementing mods. But most of the time, my mod idea is either too complex to be limited to mod hooks, or it requires modifying a specific part of code that modhooks cannot access. So expect to override functions if your mod is pretty involved.

# Baba modding random tips and tricks
## My Lua file conventions
This is just my personal convention that I use to organize my Lua files. Given that there are pros and cons to both overriding functions and modhooks + other nonconflicting methods, I find it better to organize my Lua files to distinguish between the two. You don't have to follow them, especially if you just want to make a simple mod. But if your mod grows in complexity, maybe this convention can help in organization.

- Place overwritten functions in your Lua file named exactly as the Lua file you took the function from. If you wanted to override `testcond()` from `conditions.lua`, make new file in `<your levelpack folder>/Lua` called `conditions.lua` and place it there.
- Other files in your mod should *not* contain overwritten functions. You can put anything else in them. Modhooks, custom functions, custom variables, etc. 
  - Name these files in a way such that there's little probability that another modder will have the exact filename. For example, my modpack has Lua files that start with "`th_`" as a code to indicate that these files relate to the `THIS` mod. It's a bit of a weird naming scheme, but thats why it works as a unique name.


## Printing debug from output
There are a few different methods for printing output for debug purposes. The first one is more simple:

```lua
timedmessage("message", x, y)
```
This displays "message" on screen at the specified coordinates for a few seconds. If x and y are omitted, the text will display at the top left corner. Can be used if you just want a quick output. However, printing different messages with `timedmessage()` will cause the text to be unreadable. 

A more advanced way allows you to see `print()` statements line by line in the terminal. A way to set this up:
1) Download [git-bash](https://git-scm.com/downloads)
2) Run the git-bash terminal
3) In the terminal, `cd` to where "Baba Is You.exe" is.
4) Run the command:
	```
	"Baba Is You.exe" | cat
	```
This will show a live feed of any `print()` statements the game encounters in its code and/or your code.

## Reloading your mod after making changes
Everytime you make changes to you lua files, the game will not update your changes until you reload them. The most thorough way of doing this is to restart the game. **However** if you confided your mods to a levelpack, it's more simple:

1. Assuming you are in the level editor screen: Click on *Menu -> Return to Level List -> Return to Levelpack List*
2. *Edit Levelpacks ->* Click on your levelpack

Or an even faster way:
1. Assuming you are in the level editor screen: Click on *Menu -> Return to Main Editor Menu*
2. *Edit Levelpacks ->* Click on your levelpack

The game will unload your mods the moment you go to the levelpack list menu or any menu before that. It then loads your mods the moment you select your levelpack to edit.

**Note:** when it comes to reloading *sprites* after adding them, I found that the above trick doesn't work and resort to restarting the game instead. If there's a faster way of reloading sprites, let me know.