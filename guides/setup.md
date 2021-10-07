# Setting up your enviornment
## Tools recommended
Not all of these tools are needed to start modding. These are just some tools that I find helpful when I'm doing some really complex modding. You can at least do the first bullet to get started.
- **A text editor** - This could be as simple as notepad, although I *highly* encourage a more feature-heavy text editor. I use VSCode mainly because I'm familiar with it.
- **A way to search through multiple files** - Baba Is You's codebase is (admittedly) tricky to navigate. Having a tool to scan through Baba's codebase really helps in finding function definitions and understanding the code. A feature-rich text editor (like VSCode) tends to have a built-in way to do this. Though you can also use a different tool if you prefer that.
- **git-bash or some terminal with a linux-like enviornment** - A bit more advanced to setup, but I use this because it allows me to see logging from the game via `print()`'s. (See [Printing debug from output](programming_start.md#printing-debug-from-output))
- **Git** - this is totally optional, but I find it useful to have some version control for organization. Git especially works well with VScode since it's integrated to the editor. I could keep track of my changes while I develop and use the information to narrow down bugs made from recent changes. But again, this isn't needed if you just want to make a simple mod.  

## Modding Organization/Structure
Modding can be done to affect your copy of Baba Is You globally. However, I find it neater to confide modding to seperate levelpacks. You can create a new levelpack, put your custom lua files in it, and the game will only use your custom lua scripts whenever you are playing/editing your levelpack. You can even test your mods by making levels within the levelpack.

### Setting up a levelpack for modding
1. From baba title: `Level Editor -> Edit Levelpacks -> Create a new levelpack`
2. Close the game and navigate to `<Baba game directory>\Data\Worlds\<world folder>`
    - `<world folder>` will most likely be named something like `63World` or some other number. To determine which folder is your levelpack, look in each folder for a `world_data.txt` file. Inside it, look for whatever you named your levelpack under `[General]`.
3. Edit `world_data.txt` and add `mods=1` underneath the `[General]` section.
    - `mods=1` will not be seen if you just made your levelpack.


### Folder Structure
A levelpack has a few designated folders where you can put your custom assets in. Installation of most Baba mods tends to involve copying these folders into the levelpack directory. Note that a levelpack can have some folders missing, which the game is perfectly fine with. If you have a custom asset to add, make a new folder in the levelpack with the name *exactly* as listed below. 
*Subdirectories within these folders will be ignored by the game.*
   - `Lua`
   - `Sprites`
   - `Music`
   - `Palettes`
   - `Themes`