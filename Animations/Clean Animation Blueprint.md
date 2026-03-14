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

11. Add the jump / land states

    1. Open the Main States state  machine
    2. Add a State Machine for the land animation (name it Land) and connect it with the Locomotion state machine

        ![](./Assets/Clean%20Animation%20Blueprint/Base_Structure_10.png)

    3. For the transition logic, we wantto only allow it to transition from Land to Locomotion if shouldMove is true

        ![](./Assets/Clean%20Animation%20Blueprint/Base_Structure_11.png)

    4. Inside the Land state machine, use a landing animation

        ![](./Assets/Clean%20Animation%20Blueprint/Base_Structure_12.png)

    5. Crete a State Alias (name it To Falling)

        We use a State Alias so we can group multiple state machine into 1 group and control how it can transition to another state machine

        Select:
            - Locomotion
            - Land

        ![](./Assets/Clean%20Animation%20Blueprint/Base_Structure_13.png)

        Here, it is saying that this alias represents the Locomotion and Land State Machines so any animations we connect with this alias would be able to transition between them.

    6. Add 2 new State Machines (Jump and Fall Loop) and connect it like this:
        - To Falling > Jump
        - To Falling > Fall Loop
        - Jump > Fall Loop

        ![](./Assets/Clean%20Animation%20Blueprint/Base_Structure_14.png)

    7. Use a Jump animationfor the Jump State Machine

        ![](./Assets/Clean%20Animation%20Blueprint/Base_Structure_15.png)

        For the transition logic between "To Falling" and Jump, we want to check if the character is currently falling and if the Z velocity is greater than 100 (or any value).

        ![](./Assets/Clean%20Animation%20Blueprint/Base_Structure_16.png)

        To get the Z velocity, get the Velocity variable that was created earlier and Split it to get X, Y, and Z.

        For the transition between Jump and Fall Loop, we just want it allow it without any validation checks

        ![](./Assets/Clean%20Animation%20Blueprint/Base_Structure_19.png)

    8. Use a Fall animation for the Fall Loop State Machine
        
        ![](./Assets/Clean%20Animation%20Blueprint/Base_Structure_17.png)

        For the transition logic between "To Falling" and "Fall Loop", we want to check if the character is currently falling

        ![](./Assets/Clean%20Animation%20Blueprint/Base_Structure_18.png)

12. Add a State Alias (name it To Land) and connect it with the Land state machine

    ![](./Assets/Clean%20Animation%20Blueprint/Base_Structure_20.png)

    - Select Jump and Fall Loop so the Jump or Fall Loop states can transition to Land

        ![](./Assets/Clean%20Animation%20Blueprint/Base_Structure_21.png)


    For the transition logic, we want to transition to Land if the character is not falling.

    ![](./Assets/Clean%20Animation%20Blueprint/Base_Structure_22.png)


Now this base animation blueprint is complete. The character should be able to move and jump.