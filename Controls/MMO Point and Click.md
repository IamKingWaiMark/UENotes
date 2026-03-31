# Overview

Point and Click controls like an  MMO


# Creating Game Mode

1. Create a new Blueprint Class and select Game Mode Base
2. Apply it to your project, open Project Settings > Maps & Modes > Set Default GameMode

    ![](../Controls/Assets/MMO%20Point%20and%20Click/Project%20Settings%20Game%20Mode.png)

# Creating the Player Controller

1. Create a new Blueprint Class and select Player Controller
2. Open your Game Mode and set <b>Player Controller Class</b>  with your custom Player Controller

    ![](./Assets/MMO%20Point%20and%20Click/PlayerController.png)

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

1. Create the Input Actions

    - In your Content Drawer, create a folder named Input.
    - Right-click inside the folder: Input > Input Action. Name it IA_Move.
        - Open it. Set Value Type to Axis2D (Vector2D). Save and close.
    - Right-click again: Input > Input Action. Name it IA_Look.
        - Open it. Set Value Type to Axis2D (Vector2D). Save and close.
    - Right-click: Input > Input Mapping Context. Name it IMC_Default.

2. Mapping the Keys and Mouse
    - Now we tell IMC_Default how to translate your physical hardware into those actions.
    - Open IMC_Default.
    - Click + to add a mapping and select IA_Move. Add four key slots (W, S, A, D) and apply these exact Modifiers:
        - W
            - Swizzle Input Axis Values
                - YXZ
            - Negate
        - S 
            - Swizzle Input Axis Values
                - YXZ
        - D 
            - Swizzle Input Axis Values
                - XZY
        - A 
            - Swizzle Input Axis Values
                - XZY
            - Negate

        Everytime you activate the keys in this way, you get aa single value.

        You use <b>Swizzle Input Axis Values</b> to set where that value returns to. 

        For example, IA_Move returns a Vector2DD value so you can have a X or Y. By default, X is used so you need to use the <b>Swizzle Input Axis Values</b> to say wether it returns the value to X or Y.

    - Click + to add another mapping for IA_Look.
        - Select Mouse XY 2D-Axis.
        - Add the Negate modifier. Expand it and uncheck X and Z (leave only Y checked). This ensures "Inverted Y" is off.


3. Character Blueprint and Components

    - In your Content Drawer, right-click and select Blueprint Class > Character. Name it BP_ThirdPersonCharacter.
    - Open it and go to the Components panel (top left):
        - Click Add and search for Spring Arm. Name it CameraBoom.
        - With CameraBoom selected, click Add and search for Camera. Name it FollowCamera.
    - Spring Arm (CameraBoom) Settings: * In the Details panel, set Target Arm Length to 400.0.
        - Ensure Use Pawn Control Rotation is Checked. (This allows the mouse to rotate the arm).
    - Camera (FollowCamera) Settings:
        - Ensure Use Pawn Control Rotation is Unchecked. (The camera should only follow the arm, not rotate itself).


4. The "Template" Rotation Settings

    This is the most critical step to prevent jitter and get the specific movement feel you're looking for. We need to tell the character's body to ignore the camera's rotation but follow the movement direction.

    - In the Components panel, click on the very top item: BP_ThirdPersonCharacter (Self).

        - In the Details panel, search for Use Controller Rotation Yaw. Uncheck this box. (This stops the character from snapping to face the same way as the mouse).

    - Next, click on the Character Movement (Component) in the components list.

        - Search for Orient Rotation to Movement. Check this box. (This makes the character turn to face the direction they are running).
        - Search for Rotation Rate. Set the Yaw to 500.0. (This controls how smoothly the character rotates).
        - Search for Use Controller Desired Rotation. Uncheck this box. (If this is checked, it will fight the "Orient Rotation to Movement" and cause jitter).



5. Activating the Input Mapping

    Before your WASD and Mouse inputs will work, you have to tell the player's "brain" to use the mapping context we built in Step 1.

    - In BP_ThirdPersonCharacter, click on the Event Graph tab.
    - Locate the Event BeginPlay node.
    - Drag a wire from Event BeginPlay and search for Get Player Controller.
    - Drag a wire from the Return Value of the Player Controller and search for Enhanced Input Local Player Subsystem.
    - Drag a wire from that subsystem and search for Add Mapping Context.
    - In the Mapping Context dropdown on that node, select your IMC_Default.


6. Implementing Camera-Relative Movement

    This is the exact math used in the UE5 Third Person Template to ensure that "Forward" always means "Away from the Camera" and "Backward" means "Toward the Camera," without any jitter.

    - In the Event Graph, right-click and search for IA_Move. Select the Enhanced Action Event.
    - Right-click the Action Value pin and select Split Struct Pin to get X and Y.
    - Drag a wire from Triggered and add two Add Movement Input nodes.
    - The Rotation Math:
        - Right-click and search for Get Control Rotation.
        - Drag a wire from it and search for Break Rotator.
        - Drag a wire from the Yaw pin only and search for Make Rotator. (This ignores the Pitch, so your character doesn't try to walk into the ground when you look down).
    - Directional Vectors:
        - From the Return Value of the Make Rotator, drag a wire and search for Get Forward Vector. Plug this into the World Direction of the first "Add Movement Input."
        - From that same Return Value, drag a wire and search for Get Right Vector. Plug this into the World Direction of the second "Add Movement Input."
    - Connecting the Keys:
        - Connect Action Value Y to the Scale Value of the "Forward" node.
        - Connect Action Value X to the Scale Value of the "Right" node.


7. Adding the Mouse Look logic

    This final step allows the Spring Arm (and the camera attached to it) to rotate around the character. Because of the math we did in Step 6, your WASD movement will now automatically update to match wherever you point this camera.

    - In the Event Graph, right-click and search for the IA_Look event.
    - Right-click the Action Value pin and select Split Struct Pin to get X (Mouse Left/Right) and Y (Mouse Up/Down).
    - Drag a wire from Triggered and search for Add Controller Yaw Input. Connect Action Value X to the Val pin.
    - Drag another wire from Triggered and search for Add Controller Pitch Input. Connect Action Value Y to the Val pin.


Final Verification:

    - Press W: Character runs away from the camera.
    - Press S: Character turns 180° and runs toward the camera.
    - Rotate Mouse: The camera orbits, and the character stays independent until you press a movement key.
    - No Jitter: Since we are using Get Control Rotation (which is smooth) and Orient Rotation to Movement (which handles the mesh turning), the jitter from the previous "Actor Forward Vector" method is gone.


# Configuring Jumping

1. Create an Input Action (name it IA_Jump)
2. Add it to the mappings in an Input Mapping Context
3. Open the Character Blueprint and add the IA_Jump input event
4. When triggered, called the built-in Jump function

    ![](./Assets/MMO%20Point%20and%20Click/Jump.png)


# Configuring Camera Movements


For the setup you want, use these settings on your Character:

- Character

    Use Controller Rotation Yaw = False
    
- Character Movement

    Orient Rotation to Movement = False if you do not want the character to auto-turn while moving

- Spring Arm (if your camera is attached to a spring arm)

    Use Pawn Control Rotation = True

- Camera

    Usually <b>Use Pawn Control Rotation = False</b> when the spring arm is already using it, so the boom rotates the camera for you