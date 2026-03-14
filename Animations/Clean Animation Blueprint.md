# Overview

This documentation will show you how to create an Animation Blueprint from Scratch and apply it to the Game.


# Base Structure

1. Create an Animation Blueprint
2. In the AnimGraph
    - Add a "Slot Default Slot" and connect it to the Output Pose
        - This will allow montages to play and overwrite the animations
    - Create a new State Machine (name it Main States) and connect that with the Default Slot

    ![](./Assets/Clean%20Animation%20Blueprint/Base_Structure_1.png)

    - Add another State Machine (name it Locomotion)
    - Add a "New Saved Cache Pose" (name it Locomotion) and connect it with the Locomotion State Machine

    ![](./Assets/Clean%20Animation%20Blueprint/Base_Structure_2.png)

3. Open the "Main States" State Machine

    - Add a new State (name it Locomotion)
    - Connect it with the Locomotion Cached pose

    ![](./Assets/Clean%20Animation%20Blueprint/Base_Structure_3.png)



So the flow is that it goes from Main States > Default Slot > Output pose

The "Main States" state machine just outputs what "Locomotion" is outputting 



4. Open the "Locomotion" state machine and add 2 States
    - Idle
    - Walk / Run

5. Connect the "Idle" and "Walk / Run" state machine so we can define when they can transition between the states

    ![](./Assets/Clean%20Animation%20Blueprint/Base_Structure_4.png)

6. We want to make it transition to "Walk / Run" when the character is moving and back to "Idle" when it stops moving.

    To do this, we need to create a couple of variables:

    - ShouldMove
    - GroundSpeed
    - Velocity

    Go to the Event Graph of the Animation Blueprint

    Follow the instructions of <b>Retrieving the Controlled Character</b> and <b>Getting the Forward Moving Speed</b> from [Useful Animation Blueprint Nodes](Useful%20Animation%20Blueprint%20Nodes.md) to set the GroundSpeed and Velocity variables


    For ShouldMove, just check if GroundSpeed is greater than 0.

    ![](./Assets/Clean%20Animation%20Blueprint/Base_Structure_5.png)

7. Open up the Locomotion state machine and open the transition that goes from "Idle" to "Walk / Run"

    We want it to be able to transition to "Walk / Run" if ShouldMove is true (this means that GroundSpeed is greater than 0)

    ![](./Assets/Clean%20Animation%20Blueprint/Base_Structure_6.png)

8. Open up the Locomotion state machine and open the transition that goes from "Walk / Run" to "Idle"

    We want it to be able to transition from Walk / Run back to Idle so we need to set "Can Enter Transition" when ShouldMove is false (or GroundSpeed is less than or equal to 0)

    ![](./Assets//Clean%20Animation%20Blueprint/Base_Structure_7.png)




At this point, when GroundSpeed is greater than 0, it will transition to "Walk / Run". If it is 0 or less, it will stay at "Idle" or transition from "Walk / Run" to "Idle"


9. Open the "Idle" state machine and use an Idle pose animation.

    ![](./Assets/Clean%20Animation%20Blueprint/Base_Structure_8.png)

    Apply this Animation blueprint to the character blueprint for the character and you should see the Idle pose in action.


10. Open the "Walk / Run" state machine and use a blend space for the walk / run animation

    - First create a variable called "Direction" and then follow the instructions to [Get Character facing direction](Useful%20Animation%20Blueprint%20Nodes.md)


     ![](./Assets/Clean%20Animation%20Blueprint/Base_Structure_9.png)

    At this point, you should be able to transition between idle and the walk/run animations.