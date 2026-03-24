# Overview

Focus target based on Mouse Click

# Method

The way to approach this problem is to do it in the Player Controller using the Get Hit Result Under Cursor by Channel node to detect what object the cursor is pointing at.

Make sure the object has 2 functions:

- ClearFocus
- SetFocus


The ClearFocus function should be used to clear all the indicators from the object, e.g. widget, whent he player selects a new target.

The SetFocus function should be used to set all indicators from the object when the player selects it.



# How to in the Player Controller

1. Open your Player Controller Blueprint (create one and apply it in your Game Modae if you have not done this)
2. Add a Left Mouse Button event or an Input Action, depending on your setup

    ![](./Assets/Focusing%20Objects/Left%20Mouse%20Button.png)

3. Use the Get Hit Result Undder Cursor by Channel node to detect what the mouse cursor is hitting

    ![](./Assets/Focusing%20Objects/Get%20Hit%20Result%20Node.png)

4. Split "Hit Result" to get more from the object, specifically the "Hit Result Hit Actor" which returns the actor that it first hit

    ![](./Assets/Focusing%20Objects/Get%20Hit%20Result%20Node%20Split.png)

5. Create a variable for the focus target (e.g. CurrentFocusTarget) 

    ![](./Assets//Focusing%20Objects/Varaible.png)

6. Cast the Hit Result Hit Actor to the class of your choice.

    ![](./Assets/Focusing%20Objects/Cast.png)

7. After we want:

    1. Check if the new target (Hit Result Hit Actor) is equal to the CurrentFocusTarget.
        - If NOT, we want to call ClearFocus on the old target if it is valid
        - If TRUE, we want to set the new target as the CurrentFocusTarget and call the SetFocus function from it.


    ![](./Assets/Focusing%20Objects/SetFocusTarget.png)

