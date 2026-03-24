# Overview

Point and Click controls like an  MMO


# Creating Game Mode

1. Create a new Blueprint Class and select Game Mode Base
2. Apply it to your project, open Project Settings > Maps & Modes > Set Default GameMode

    ![](../Controls/Assets/MMO%20Point%20and%20Click/Project%20Settings%20Game%20Mode.png)


# Creating the Character Blueprint

1. Createa new Blueprint Class and select Character
2. Open the Character blueprint, you should see these components already added:

    ![](../Controls/Assets/MMO%20Point%20and%20Click/Character%20Component%20BP.png)

    - Capsule Component: the collision body for the character

    - Mesh: the visual skeletal mesh for the character model

    - CharacterMovement: handles walking, jumping, falling, movement settings, and related behavior

3. Connect it to the Game Mode, open the Game Mode and set it as the Default Pawn class in the Details panel.

    ![](../Controls/Assets/MMO%20Point%20and%20Click/Default%20Pawn%20Class%20.png)



# Configuring WASD Keys for Movement


1. Create an Input Action - name it IA_Move, Type: Axis2D
    
    Because WASD movement is not a single on/off input. It represents two numbers at once:

    - X = left / right
    - Y = forward / back

    So IA_Move is set to Axis2D because Unreal treats it as a 2D vector value (Vector2D), not a boolean. Epic’s Enhanced Input docs describe Input Actions as supporting different value types, including Axis2D for two-dimensional input.

2. Create an Input Mapping Context and add IA_Move to the mappings as add the WASD keys.

    ![](./Assets/MMO%20Point%20and%20Click/WASD%20Config%201.png)


    In an Axis2D move action, Unreal expects:

    - Y = forward/back
    - X = right/left

    For keyboard keys, each key starts as a 1D input, so you use Modifiers to place that value onto the correct axis and flip it when needed. This is where the Swizzle Input Axis Values modifier comes it. It takes that single value as output it as an Axis value of that order.

    For example if Swizzleis set to YXZ, Y would receive that single value. This way, when you call the IA_Move input action node, you can get the X or Y values from the pin.

    ![](./Assets/MMO%20Point%20and%20Click/IA_MOVE.png)




    Set the 4 mappings like this:

    - W

        Keep key as W

        Add modifier: Swizzle Input Axis Values

        Set swizzle to YXZ

        This turns the key’s value into (X=0, Y=1), so W becomes forward.

    - S

        Keep key as S

        Add modifier: Swizzle Input Axis Values

        Set swizzle to YXZ

        Add modifier: Negate

        This becomes (X=0, Y=-1), so S becomes backward.

    - A

        Keep key as A

        Add modifier: Negate

        This becomes (X=-1, Y=0), so A becomes left.

    - D

        Keep key as D

        No modifier needed

        This stays (X=1, Y=0), so D becomes right.


# Moving the Camera on Mouse click