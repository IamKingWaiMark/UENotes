# Tutorial

https://www.youtube.com/watch?v=HRagD5L-WF8


# Niagara System
1. Create a Niagara System
2. Add a Minial Emitter
3. Change the Renderer to a Mesh Renderer (Default is Sprite Renderer)
4. Add a Spawn Rate module in the Emitter Update
    - Set:
        1. Spawn Rate: 20
5. Go to the Initialize Particle section and set
    - Lifetime Mode: Random
    - Lifetime Min: 1.2
    - Lifetime Max: 1.6
    - Mesh Scale Mode: Random Uniform
    - Mesh Uniform Scale Min: 1
    - Mesh Uniform Scale Max: 2

6. Add a Shape Location module to spawn the particles around that area in the Particle Spawn section
    - Set:
        1. Shape Primitive: Cylinder
        2. Cylinder Height: 0
        3. Cylinder Radius: 20
7. Add a Velocity Module in the Particle Spawn section to give it some motion
    - Set:
        1. Velocity Mode: In Cone
        2. Velocity Speed: 800
        3. Cone Axis: 0, 0, 1 (this makes it go up)

8. [Create Mesh for Smoke](#create-mesh-for-smoke)

9. Set the mesh for the Mesh Renderer

    ![](./Stylized%20Smoke/StaticMesh.png)

10. Add a initial Mesh Orientation module to Particle Spawn to change the orientation when the mesh spawns

    - Set:
        1. Mesh Orientation Mode: Random

11. Control the scale of the meshes by adding a Scale Mesh Size Module in Particle Update
    - Set
        1. Scale Factor: Vector from Float 

        ![](./Stylized%20Smoke/ScaleMeshSize.png)

        2. Value: Float from Curve

        ![](./Stylized%20Smoke/ScaleMeshSizeCurveGraph.png)

        3. Key:
            - 1: 0.0,0.1
            - 2: 1.0, 1.5

        4. Select both Keys, right click and select Auto to create a curve instead of a linear line
            


    This will allow us to modify the value using a curve graph

12. [Creating the Smoke Material](#creating-the-smoke-material)

13. Enable Material Overrides for the Mesh Renderer
    - Add and assign the Smoke material

14. Animate the smoke
    - Add a Dynamic Material Parameters module to the Particle Update
    - Conver the Erosion to a Curve
        - Key 0: 0.3, 15
        - Key 1: 1, 0

15. Add a Scale Color module to change the color transition in the Particle Update
    - Set:
        1. Scale Mode: RGBATogether
        2. Convert Scale RGBA to Curve

    ![](./Stylized%20Smoke/ScaleColor.png)

16. Add a Wind Force module in the Particle Update to add wind effects
    - Play with the values


# Create Mesh for Smoke

1. Open Bllender
2. Select everything and delete the default stuff
    - "A"
    - "Delete"
3. Add a Ico Sphere
    - "Shift" + "A" > Mesh > Ico Sphere
4. Set subdivision to "1" with the menu at the bottom right when you added the mesh

    ![](./Stylized%20Smoke/IcoSphere.png)

5. With the object selected, open the Modifier section with the "Wrench" button in the toolbar on the right 
    - Add a Subdivision modifier

        ![](./Stylized%20Smoke/IcoSphereSubdivisionMod.png)

6. Duplicate the mesh a couple of times and form a cloud like mesh

    ![](./Stylized%20Smoke/Smoke_Mesh.png)

7. Select all the meshs and join them to 1 mesh
    - "Ctrl" + "J"

8. Add a Remesh modifier to remove unwanted overlapping veritices

    - Set:
        1. Voxel
        2. Voxel Size: 0.15m
        3. Adaptivity: 0
        4. Smooth Shading to smooth the object

9. Apply all the modifiers

    ![](./Stylized%20Smoke/ApplyMods.png)

10. Add a Decimate modifier to reduce the poly count and then apply it

    ![](./Stylized%20Smoke/Decimate.png)

    - Set:
        1. Un-Subdivide
        2. Iterations: 2

11. Fix the UVs

    1. Open a new window, hover the mouse over to the top left corner of the viewport window and drag (You need to see a crosshair cursor)

        ![](./Stylized%20Smoke/SplitWindow.png)

    2. Open the UV editor in the new window

        ![](./Stylized%20Smoke/UVEditor.png)

    3. In the viewport with the object, switch to Edit Mode and select all the veriticies
        - Right now there is no UVs

    4. Unwrap the UVs by pressing "U" and selecting "Sphere Projection"

        ![](./Stylized%20Smoke/SphereProjection.png)

    5. Resize the UVs to make it fit in the map

    6. Export as FBX and Import to UE
        - Make sure the Static Mesh is there when imported
        - Check the Static Mesh tab on the popup window



# Creating the Smoke Material

1. Create a new Material
    - M_Smoke

2. Switch the Blend Mode to "Mask"
3. Add a TextureSampleParameter2D node for the main texture
    - MainTex
4. Connect the RGB to the Power node
5. Add a Particle Color node so the Material color can be modified with the particle color in the Niagara System
    - Multiply the result of the Power Node and the RGB result from the Particle Color
    - Connect that result to the Emissive pin for the Material

    ![](./Stylized%20Smoke/SmokeTexture_1.png)

6. Add a Dynamic Parameter node
    - The Dynamic Parameter node will allow you to modify the material values in the Niagara System through the Dynamic Parameter module that you can add to the Emitter
    - Set:
        - Index[0]: Erosion
        - Index[1]: Power

        - Default Values:
            - R: 20 (Erosion)
            - G: 0 (Power)

7. Connect the Power parameter with the Exp pin of the Power Node
8. Multiply the R value of the MainTex node with the Erosion
9. Connect the result of the Erosion Multiplier with the Opacity Mask

    ![](./Stylized%20Smoke/SmokeTexture_2.png)

10. Change the texture of the MainTex node with your texture

11. Optionally, you can create parameters for the Metali and Specular outputs with paraeters to modify it

    ![](./Stylized%20Smoke/SmokeTexture_3.png)

12. Create a material instance