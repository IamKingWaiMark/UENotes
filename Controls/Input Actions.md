# Overview

This documentation explains and show you how to use Input Actions.


# Definition and Usage

In Unreal Engine, Input Actions are named input events you create to represent something the player wants to do, like Jump, Attack, or Interact.

Instead of checking a specific key directly everywhere, you make an Input Action once, then bind keys, mouse buttons, or controller buttons to it. That makes your controls easier to manage and change later.


To use Input Actions, first you would need an Input Mapping Context.

Then you add the Input Actions in it like so:

![](./Assets/Controls/Input%20Mapping%20Context.png)

You can set which buttons will trigger it after adding the Input Actions.

Once you have the Input Mapping Context, use it by using "Get EnhancedInputLocalPlayerSubsysten" from Local Player Subsystem category and "Add Mapping Context" to apply it to your game controller

![](./Assets/Controls/Input%20Mapping%20Context%20With%20Subsystem.png)

Once you have connected your Input Mapping Context, you can use the Input Action in your blueprints by typing the Input Action name, e.g.  IA_Aim.

![](./Assets/Controls/Input%20Action%201.png)