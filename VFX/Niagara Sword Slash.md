# Tutorial

https://www.youtube.com/watch?v=djlnnPvFR0Q&list=PLpPd_BKEUoYhyGZHSK1pUMOsy75iNE7_Z&index=3


# Steps 

1. Create a new Niagara System
    - NS_SwordSlash

2. Add a Emitter
    - SlashMeshBright01

3. Add a SpawnBurstInstantaneous Module to Emitter Update
4. Remove the Sprite Renderer and add a Mesh Renderer

5. [Create the Slash Mesh In Blender](#create-the-slash-mesh-in-blender) and use it in the Renderer

6. Set the Mesh Renderer to use the Slash mesh

7. Set the Lifetime to .35 for the Initialize Particle section, since it will end quickly

8. Optionally, off set it, 75, 0, 150 (X, Y, Z)

9. Set Mesh Scale Mode: Uniform
    - Mesh Uniform Scale: 0.75

10. Rotate it when it spawns by using the Initial Mesh Orienttion module in Particle Spawn
    - Set:
        1. Mesh Orientaation Mode: None
            - This is so you can specify the rotation
        2. Set rotation to .25 in the X-axis

        ![](./Swortd%20Slash/InitialMeshOrientation.png)

11. In Particle Update, add the Mesh Rotation Force to rotate the object during updates 
    - For Rotation, make a vector and set Y as a curve

        ![](./Swortd%20Slash/RotationForceY.png)

    - Update the 2nd key to (0.75, -0.25)
    - Select all the keys and make it a Auto curve
    - Play with the values
        - Set Scale Curve: 250 - speeds up

12. [Creating the Material](#creating-the-material)

13. Apply the Slash material to the Mesh in the Mesh Renderer and play with the values

13. Set the color in Initialize Particle

14. Set to Local Space for the Emitter so we can move it around

15. Duplicate the Emitter and name it
    - SlashMeshBright02

    - Set the Lifetime: 0.37 to last longer
    - Color: 75, 8, 1

16. Duplicate the Emitter and name it
    - SlashMeshBright03

    - Set the Lifetime: 0.39 to last longer
    - Color: 0, 0, 0
    - The Black is not showing because the material is Additive.
        - Duplicate the Material Instance and set Blend Mode: Translucent
        - Apply it  

    ![](./Swortd%20Slash/Slash3MaterialInstance.png)


# Create the Slash Mesh In Blender


1. Add a Cylinder
2. Set CapFill Type: Nothing to remove the top and bottom faces
3. In Object Mode, right click on the object > Shade Smooth to smooth the object's edges
4. Go to Edit Mode and add 3 Edge Loops (create sections)
    - Select the object
    - Press "Ctrl" + "R", then Scroll to create the 3 edge loops

    ![](./Swortd%20Slash/EdgeLoops.png)
5. Select all the edges in the center with "Shift" + "Alt" + "A", then click on the center

    ![](./Swortd%20Slash/EdgeLoopCenter.png)
6. Turn on "Proportion Editing" by pressing "O"
7. Create a UFO Shape object

    ![](./Swortd%20Slash/UFO.png)

8. Flatten it by selecting everything ("A"), then scale ("S"), lock to Z-axis ("Z") and move the mouse to flatten

    ![](./Swortd%20Slash/FlatUFO.png)


9. Export as FBS and import


# Creating the Material

1. Create a Material
    - M_Slash

2. Set Blend Mode: Additive

3. Add a TextureSampleParameter2D node for the Texture
    - Name: Vo
    - Set the texture

4. Duplicate the TextureSampleParameter2D node for the mask
    - Name: Mask
    - Set the Texture for the mask, in the example, it is using a circle to mask the circle object

5. Add a Particle color node and multiply it with the mask to apply it and plug it into the Emissive Color

    ![](./Swortd%20Slash/SwordSlashMask.png)

6. Create a Dynamic Parameter so we can modify the material values in the Niagara System
    - Set Param names:
        - [0] Power
        - [1] Erosion

7. Combine the RGB of the slash texture with the Power pin using a Power node
    - Dissolves the material

8. Multiply the RGB of the Mask wit the result of the Power node


![](./Swortd%20Slash/SlashTexture.png)


9. Make the texture movable 

![](./Swortd%20Slash/Movable.png)

10. Convert the mast material into a Material Instance