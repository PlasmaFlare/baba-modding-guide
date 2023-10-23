# First Steps
## Tools recommended
Not all of these tools are needed to start modding. These are just some tools that I find helpful when I'm doing some really complex modding. You can at least do the first bullet to get started.
- **A text editor** - This could be as simple as notepad, although I *highly* encourage a more feature-heavy text editor. I use VSCode mainly because I'm familiar with it.
- **A way to search through multiple files** - Baba Is You's codebase is (admittedly) tricky to navigate. Having a tool to scan through Baba's codebase really helps in finding function definitions and understanding the code. A feature-rich text editor (like VSCode) tends to have a built-in way to do this. Though you can also use a different tool if you prefer that.
- **git-bash or some terminal with a linux-like enviornment** - A bit more advanced to setup, but I use this because it allows me to see logging from the game via `print()`'s. (See [Printing debug from output](programming_start.md#printing-debug-from-output))
- **Git** - this is totally optional, but I find it useful to have some version control for organization. Git especially works well with VScode since it's integrated to the editor. I could keep track of my changes while I develop and use the information to narrow down bugs made from recent changes. But again, this isn't needed if you just want to make a simple mod.  

## Important directories
All baba modding is done within the `Data` folder at whereever the game is installed. If you have the game on Steam:
- **Windows** - Right click the game in your library, then *Manage -> Browse Local Files*. Navigate to `Data` folder
- **Mac** - Right click the game in your library, then *Manage -> Browse Local Files*. Right-click on `Baba Is You.app` and pick *Show Package Contents*. Navigate to `Content/Resources/Data`
- **Linux** - Same as Windows

### `Data` folder contents
You'll see a bunch of lua files and various other folders. The lua files contain most of the logic that makes the game work. Feel free to [familiarize with some of these files](../references/lua%20files.md) when you get to modding. But for now you can ignore them.

The `Worlds` folder contains all levelpacks that you currently have. Each subfolder within `Worlds` is its own levelpack. Levelpacks are special in that any assets you put into a levelpack will be confined and only used whenever the levelpack is loaded. This makes levelpacks a good testing ground for you mods. You can create a new levelpack, put your custom lua files in it, and freely test your mod without affecting other levelpacks or the base game. (See [Setting up a levelpack for modding](#setting-up-a-levelpack-for-modding))

### Levelpack Folder Structure
A levelpack has a few designated folders where you can put your custom assets in. Installation of most Baba mods tends to involve copying these folders into the levelpack directory. Note that a levelpack can have some folders missing, which the game is perfectly fine with. If you have a custom asset to add, make a new folder in the levelpack with the name *exactly* as listed below. 
*The game will not look in subdirectories for assets.*
   - `Lua`
   - `Sprites`
   - `Music`
   - `Palettes`
   - `Themes`

## Setting up a levelpack for modding
1. From baba title: `Level Editor -> Edit Levelpacks -> Create a new levelpack`
2. Close the game and navigate to `<Baba game directory>\Data\Worlds\<levelpack folder>`
    - `<levelpack folder>` will most likely be named something like `63World` or some other number. To determine which folder is your levelpack, look in each folder for a `world_data.txt` file. Inside it, look for whatever you named your levelpack under `[General]`.
3. Edit `world_data.txt` and add `mods=1` underneath the `[General]` section.
    - `mods=1` will not be seen if you just made your levelpack.

