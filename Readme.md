# Disclaimer
## THIS IS NOT A COMPREHENSIVE GUIDE TO BABA MODDING

The game's source code has too many systems layered over one another to all fit in this modest guide. Very often you will want to get your hands dirty and analyze through the lua files yourself. But for the purpose of getting started,
this guide aims to cover some of the basic concepts that you will find common/fundamental among the game's lua files. Anything else specific, you'll have to look at the game files yourself.

This guide will assume a few things:
- You have a basic knowledge of Lua.
- You are on Windows (I don't have Mac or Linux so some directory paths might be different)
- You have the lastest Baba version on Steam

---
# Table of Contents (roughly)

- Setting up enviornment for modding
  - levelpack setup
  - Methods of adding your own code
    - Override method
    - mod hook functions
    - My personal code structure
- Level Editor Object properties brief walkthrough
- Adding a custom object to the level pallete
- Units
- Adding your own property/Getting objects with a certain property
- Brief descriptions of main game lua files


# Getting started on creating your own property
One thing to understand is that there isn't a notion of "you need to register this property first before you can define what it does". Any text object can be a property as long as its text type is set to 2. If you name a text object `text_abcd` and set its text type to 2, the game will see that there's a property called `abcd` and add that to its rule tables. So without having to define something in code, you can make the game recognize any name for a property.

As for getting which objects are affected by your property, the functions in `features.lua` handle most of this. An overview of some of these functions:
- hasfeature
- hasfeature_count
- findfeature
- findfeatureat
- findallfeature
- getunitswitheffect/getunitswithverb/getunitverbtargets?

All of these functions base their results from a global table called `featureindex`, which gets repopulated with parsed rules whenever code() gets called. How `featureindex` gets populated is beyond the scope of this guide, as it involves diving into how rule parsing works. Just know that `featureindex` contains the parsed rule data.

When implementing your own property, you would ideally use the functions in `features.lua` in different areas of the basegame code that is relevant to your property (or even in your own functions).