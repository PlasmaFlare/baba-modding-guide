# Implementing a custom property
### How to detect if my property is active?
You added your custom property to the editor (as shown in [this guide](custom_word_start.md)). Now how do you detect whether or not it is active? This is handled by the functions in `features.lua`. Most of these functions specialize in returning units that are affected by a certain property. An overview of some of these functions can be found [here](../references/rules.md#featureslua-functions). When implementing your own property, you would ideally use these functions in different areas of the basegame code that is relevant to your property (or even in your own functions).

### Where to detect my property?
This involves more research on your part, the idea being that you can look at how existing words are implemented in order to base your implementation. The Baba wiki has a few excellent pages for describing the execution order of some of the game's main functions.
- A brief overview can be found here: https://babaiswiki.fandom.com/wiki/Advanced_rulebook
- A more detailed explaination can be found here: https://babaiswiki.fandom.com/wiki/Order_of_Operations


### Alright, I figured out where and how to check for my property. How do I go about implementing it?
From here on out, it really depends on what your property does. Some general advice:
- Make sure you have a clear idea on what your property should do. Consider possible edge cases and how the property should behave.
- Research. You can use this guide as an aid for basic concepts. But don't be afraid of looking through the source code. 
- Look into how existing properties implement certain aspects of your property. 
    - If for instance I wanted to make a property that destroys texts blocks while spawning letters in its place, I would research on how "defeat" destroys "you" objects and "more" spawns new objects.
    - If there are no existing properties that do this, you'll need to do some further digging. Look at the [References section](../readme.md) for some pages/links detailing common concepts in the game's code. And as always, don't be afraid to look at the source code is well!
- Form a broad picture of what your implementation would look like. 
  - Which functions would you need to override or which modhooks would you need to use? 
  - Would you need to update code of existing properties to make your property compatable?
  - Do you feel that your code would be too hard to manage? 
  - Find ways to reduce the complexity if needed, either by finding workarounds or simplifying how you want your property to act. When its at a level that you are confident in doing, then you can start implementing.






