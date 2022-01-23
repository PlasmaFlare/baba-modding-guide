# Baba Modding Guide
This is a knowledge dump of some basic concepts I know from modding Baba Is You. Consider this purely experimental, as it's difficult cover a topic like Baba modding when I'm not the main developer of the game and not a lot of documentation is avaliable. I'm not even sure if this "wiki-like" format is the best method for explaining modding information. But we'll see how this goes.

## Disclaimer
I want to emphasize on one thing: **do not expect this to be a comprehensive tutorial for modding Baba.** Instead, use this as another source of reference when diving into the source code yourself. I do not intend this guide to walk you through everything as the game's source code is too complex for this modest guide to cover. When you're just starting out, a lot of active research is involved, and very often you'll need to look at the source code to understand how the game works. You will be confused and overwhelmed at times, but it is a necessary evil if you want to mod Baba Is You.

With that being said, this guide tries to alleviate some of the confusion you might get from examining the source code, as well as provide small tips for organizing your code effectively. Use this guide to help clarify things, so that you can work towards implementing your mod. Feel free to correct me if you find any errors. You can also suggest to me what other topics aside from the ones below I should cover. But I might or might not decide to do them depending on my knowledge on the topic, or if it's too complex/specific to general Baba modding.

This guide will assume a few things:
- You have a basic knowledge of Lua.
- You are on Windows (I don't have Mac or Linux so some directory paths might be different)
- You have the latest Baba version on **Steam**

---
# Table of Contents

## Guides
(In recommended reading order)
- [Getting Started](guides/setup.md)
- [Basics of using Lua to mod Baba Is You](guides/programming_start.md)
- [Adding a custom object to the level pallete](guides/adding_obj_to_editor.md)
- [Getting started on creating a custom word](guides/custom_word_start.md)
  - [Implementing a custom property](guides/adding_properties.md)
  - [Implementing a custom condition](guides/adding_conditions.md)
## References
- [Brief overview of the game's Lua files](references/lua%20files.md)
- [Common object properties](references/editor_obj_settings.md)
- [Units](references/units.md)
- [Rules](references/rules.md)
- [Major Function Call Order](references/order%20of%20operations.md)
- [Undo System](references/undo%20system.md)
- [How Rule Parsing Works (WARNING: Advanced)](references/parsing.md)
- [Order of Operations (link to baba wiki)](https://babaiswiki.fandom.com/wiki/Order_of_Operations)
- [MMF Functions (courtesy of Hempuli)](references/MF_documentation.txt)