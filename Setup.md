# Enviornment/Structure
## Tools recommended
Not all of these tools are needed to start modding. These are just some tools that I find helpful when I'm doing some really complex modding. At most you can just start with the first bullet to get started.
- **A text editor** - This could be as simple as notepad++, although I *highly* encourage a more feature-heavy text editor. I use VSCode mainly because I used it enough to be productive.
- **A way to search through multiple files** - Baba Is You's codebase is (admittedly) tricky to navigate. Having a tool to scan through Baba's codebase really helps in finding function definitions and understanding the code. A feature-rich text editor (like VSCode) tends to have a built-in way to do this. Though you can also use grep if you prefer that.
- **git-bash or some terminal with a linux-like enviornment** - A bit more advanced to setup, but I use this because it allows me to see logging from the game via `print()`'s.
  - This can be done if you open a terminal and `cd` to where Baba is installed. Then run:
    ```
    "Baba Is You.exe" | cat
    ```
    This would start Baba normally, but in the terminal you can see output from `print()` statements. 
  - *(Note: if there are other ways to also see logging from `print()`, let me know and I can possibly update this guide.)*

## Folder structure <@Might change title>
Modding can be done to affect the entire game. However, I find it neater to confide modding to seperate levelpacks. You can create a new levelpack, put your custom lua files in it, and the game will only use your custom lua scripts whenever you are playing/editing your levelpack. You can even test your mods by making levels within the levelpack.

### Setting up a levelpack for modding
1. From baba title: `Level Editor -> Edit Levelpacks -> Create a new levelpack`
2. Close the game and navigate to `<Baba game directory>\Data\Worlds\<world folder>`
    - `<world folder>` will most likely be named something like `63World`. To determine which folder is your levelpack, look in each folder for a `world_data.txt` file. Inside it, look for whatever you named your levelpack under `[General]`.
3. Edit `world_data.txt` and add `mods=1` underneath the `[General]` section.
    - `mods` will not be seen if you haven't configured your levelpack to enable modding.


### Folder Structure
A levelpack has a few designated folders where you can put your custom assets in. Installation of most Baba mods tends to involve copying these folders into the levelpack directory. Note that a levelpack can have some folders missing, which the game is perfectly fine with. If you have a custom asset to add, just make a new folder in the levelpack with the name *exactly* as listed below:
   - `Lua`
   - `Sprites`
   - `Music`
   - `Palettes`
   - `Themes`

They're pretty self-explainatory on which assets goes where. Note that as of (10/4/21), subdirectories within these folders will be ignored by the game.

## Getting Started on Baba Programming
Programming anything to modify the game starts with making a lua file. Create the lua file in `<your levelpack folder>/Lua` and name it whatever you want (as long as it ends with `.lua`). You can make as many lua files you want. The game will load the lua files in alphabetical order.

There are two main ways to mod the game:
### Override Main Game Functions
You can directly override existing functions from the game's lua files to get the most flexibility when implementing your mods. The downside of this is that you have to handle if a Baba update changes the overwritten function, or if you want to merge two mods together.

To do this, copy the entire lua function into your own lua file. Then you can modify your copy however you like. When you load your modded levelpack, the game will use your version instead of its own, as long as you are in the modded levelpack.

### Mod hooks
To try to reduce the amount of conflicts between two different mods, Hempuli defined a small mod hook system. This works by inserting your custom function into different lists of functions. These lists of functions will then be called by the game, in the order the functions were inserted, at certain times in the game's code.

Here's an example of adding a custom function to a mod hook that executes when the level starts (taken directly from `<baba install dir>\Data\Worlds\debug\Lua\tutorial.lua`)

```
table.insert(mod_hook_functions["level_start"],
	function()
		timedmessage("Starting a new level!")
	end
)
```
The full list of modhooks is defined in `modsupport.lua`. In the above code, replace `level_start` with whichever modhook you want.

### Other notes:
You can also