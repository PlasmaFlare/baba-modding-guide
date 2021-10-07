# Getting started on creating your custom word
If you haven't already, I recommend reading the [basics on modding the game with Lua](programming_start.md).

## Step 1: Adding your custom word to the editor
---
If you are new to Baba modding without much knowledge on how the game works, you might be thinking "Why is Step 1 already adding the custom word when it's not even programmed?" Baba is programmed in a way that allows adding a new text block without needing to program its effect. When you just add the text block to a level, using it in a rule will do nothing since you haven't implemented its effect. But the game won't complain at you for not providing the functionality. This means you can add as many text blocks you want without any consequence, which is useful for testing the visuals. Later on, you can program in their individual effects.

There are two methods for adding custom text to the editor: the programmatic way and the manual way. The programmatic way is mainly covered [here](adding_obj_to_editor.md). It should ultimately be used if you are distributing your mod. But it requires that you have the right sprites for your property. 

If you just want to quickly test your property without needing custom sprites, you can do the manual way. In the level editor palette do the following:
1) Add a random text object that you won't use
2) Set its Text Type to "You" (See [Text Types](../references/editor_obj_settings.md#text-type)).
3) Name the object `text_<customword>`

So if you name a text object `text_abcd` and set its text type to "you", the game will see that there's a property called `abcd` and add that to its rule tables when the property is active. But it won't do anything else since you haven't modified the code to detect for the property `abcd`.

## Step 2: Handling your custom word
---
All custom words go through Step 1 to get the game to recognize your custom word. How to handle implementing its effects depends on the word type (Is it a property? Condition? Verb? Etc).

(Author's Note: I only have sections for property and conditions. But as a general rule of thumb, always look for how an existing word is implemented to base your code on)

- [Implementing a custom property](adding_properties.md)
- [Implementing a custom condition](adding_conditions.md)