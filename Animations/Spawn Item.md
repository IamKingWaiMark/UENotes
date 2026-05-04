# Overview

Spawning items with an drop animation.


1. Create a Actor blueprint for the item (BP_Item)
2. Open the Actor blueprint (BP_Item) and add a Static Mesh component
    - Enable Simulate Physics on the mesh component
    - Disable Collision (if needed)
    - In Begin Play, add an impulsenode to laucnh it upward and outward

        - Add Impulse node → Vector like (100, 0, 400) scaled by mass.

            ![](./Assets/Spawn%20Item/AddImpulseNode.png)

3. Spawn the actor using the Spawn Actor from Class node whenever you want, e.g. enemie is dead.

    ![](./Assets/Spawn%20Item/SpawnActor.png)