# Overview

Equiping weapons using sockets.



# Attaching a Socket

1. Open the character's Skeleton
2. On the Left panel where all the bones are listed, add a socket to where you want to equip a weapon. E.g. (RightHandSocket)

    ![](./Assets/Equip%20Weapons/Socket%20Example.png)


# Attaching a Mesh to the Socket

1. From the Blueprint Editor, get the reference to the Skeletal Mesh
2. Use the Attach Actor to Component node

    ![](./Assets/Equip%20Weapons/Attach%20Actor%20to%20Component%20Node.png)

3. Set Location Rule, Rotation Rule, and Scale Rule to <b>Snap to Target</b>

    Location Rule

        Keep Relative = keep its current relative offset

        Keep World = stay where it already is in the world

        Snap to Target = move exactly onto the parent/socket location

    Rotation Rule

        Keep Relative = keep its local angled offset relative to the hand

        Keep World = keeps facing the same world direction when attached

        Snap to Target = rotates to exactly match the hand socket

    Scale Rule

        Keep Relative → child keeps its relative scale, so it may become larger in world space because of parent scaling

        Keep World → child stays the same visible size, and Unreal adjusts relative scale to compensate

        Snap to Target → child matches the parent’s scale


4. Set the Target. This is the weapon itself.

    - Use the Spawn Actor From Class node
        - Set the Class to the Weapon / Equipment
        - Right-click on the Spawn Traansform pin and select "Split Struct Pin" to give it a defult required value
        - Make sure the weapon / equipment has no collision so it does not effect the character / mesh

    ![](./Assets/Equip%20Weapons/Spawn%20Actor%20from%20Class.png)

