# Tutorial

https://www.youtube.com/watch?v=graozMcShMA&list=PLpPd_BKEUoYhyGZHSK1pUMOsy75iNE7_Z

# Waterfall

1. Create a Niagara System and add a Minimal Emitter (name it WaterCylinder01)
2. Spawn a particle, go to Emitter Update > Add a Spawn Burst Instantaneous module
3. Remove the Sprite Renderer and add a Mesh Renderer

    ![](./Anime%20Waterfall/WaterCylinder_01_MeshRenderer.png)

4. Create a Cylinder (In Blender)
    - Clean project
    - Shift + A > Mesh > Cylinder
    - Select the Cylinder and at the bottom right menu, set Cap Fill Type to "Nothing"
        - This will not fill the top and bottom faces

            ![](./Anime%20Waterfall/Cylinder_blender_01_Cap_Fill_Results.png)

    - Go to Edit Mode with the Cylinder selected
        - Press "A" to select everything (verticies, faces, etc.)
        - Press "G" to move it
        - Press "Z" to only move in the Z axis
            - Move it above the origin floor (Press 1)
        - Swith to Object Mode, right click on the Cylinder > Shade Smooth to smooth the edges

            ![](./Anime%20Waterfall/Cylinder_No_Smooth.png)

            ![](./Anime%20Waterfall/Cylinder_Smooth.png)

    - Export it
        - File > Export > FBX 
            - Limit to :"Selected Objects"

5. Go back to the Niagara System and add the Cyclinder mehs to the Mesh Renderer
    
    ![](./Anime%20Waterfall/Mesh_Renderer_Added.png)

6. Go to the Initialize Particle seciton, set:
    - Lifetime : 5
    - Mesh Attributes > Mesh Scale Mode : Non-Uniform
        - Mesh Scale : 0.5, 0.5, 5 (X, Y, Z)
            - This streches the mesh

        ![](./Anime%20Waterfall/Stretch_Cylinder.png)

7. Set a Stylized Material, click on the Mesh Renderer module and Enable Material Override
    - Add a new Element for Override Materials
        - Create a material
            - Right click in the content browser > select Material
                - Name it, M_SolidColor
            - Set:
                - Blend Mode: TranslucentGreyTransmittance
                - Shading Model: Unlit
                    - This prevents light from affecting the material
            - Add a Particle Color node
                - This allows a Niagara System to control the color of the material
            - Connect the RGB to "Emissive Color"

            ![](./Anime%20Waterfall/ParticleColor_Material.png)

        - Set this material to the Override Material slot

            ![](./Anime%20Waterfall/AppliedSolidColorMaterial.png)

        You will now be able to set the color in the Initialize Particle section

8. Go to the Initialize Particle section, set
    - Color Mode: Direct Set
    - Color: 0, 0.4, 1


# Create the Erosion Material

1. Create a new Material
    - Name it, M_Erosion
2. Set:
    - Blend Mode: Masked
    - Shading Model: Unlit
3. Add a Texture Sample Parameter 2D node
    - Name it MainTex
    - Set the Texture
    - Connect the RGB to Emissive Color
4. Add a ParticleColor node so the Niagara System can control it
5. Multiple the ParticleColor with the SampleParameter2D

    ![](./Anime%20Waterfall/M_Erosion_01.png)

6. Add a Dynamic Parameter node so we can control the amount of Erosion
    - Set param names
        - [0] Erosion
        - [1] Power

7. Connect the SampleParameter2D RGB pin to a Power node
    - Then connect the Power parameter from the Dynamic Parameter node to the 2nd pin of the Power node
    - Then connect the result of the Power node with the B pin from Multiply

    ![](./Anime%20Waterfall/M_Erosion_Power.png)

8. Multiply the R pin from the MainTex node and multiply it with the Erosion from the parameter
    - After, connect it to the opacity mask

        ![](./Anime%20Waterfall/M_Erosion_OpacityMask.png)


        Results


        ![](./Anime%20Waterfall/M_Erosion_Results.png)

9. Set the Parameter Index to 1 for the Dynamic Parameter
    - This represents the 2nd channel


10. Create another Dyanmic Parameter, this will be used to add motion to the material
    - Set the Parameter Index to 0 
    - Param names:
        - [0] TextureTilingX
        - [1] TextureTilingY
        - [2] SpeedX
        - [3] SpeedY

11. Add a TextureCoordinate node
    - This is the UV of the cylinder
    - Append TextureTilingX and TextureTilingY
    - Multply the results of the Append node with the TextureCoordinate

12. Add a Time Node
    - This is used to animate the texture pattern vertically or horizontally
    - Add 2 Multiply node
        - Connect SpeedX with the 1st Multiply node pin
        - Connect SpeedY with the 2nd Multiply node pin
    - Connect the Time pin with the 2nd pin of both Multiply node
    - Add a AppendMany node and plug in 
        1. the Multiply result of SpeedX in R of it
        2. the Multiply result of SpeedY in the G of it

13. Connect the result of TextureCoordinate Multiplication and the RG result of the AppendMany with the Add node


![](./Anime%20Waterfall/M_Animation_Setup.png)

14. Connect the result of everythig to the UVs pin of the MainTex Texture Sample Parameter 2D node


![](./Anime%20Waterfall/M_Animation_UV.png)


15. Right click on the Material and create an Material Instance of it


NOTE: The Dynamic Parameters and the Particle Color nodes will allow the Niagara System to control the values
NOTE: Particle Color is affected by the Color of the particle in the Niagara System (e.g. inside Initialize Particle)


# Using the Erosion Material

1. Go back to the Niagara System
2. Duplicate the Emitter (WaterCylinder01) and name it WaterCylinder02
3. Apply it with the Material Override 

    ![](./Anime%20Waterfall/M_Cylinder_02.png)

4. Clickk on the "Person" button on the top of the Emitter node to isolate the scene to preview it

    ![](./Anime%20Waterfall/Emitter_Cylinder02_Isolation.png)

5. Disable the Isolation and set the Erosion Emitter to be slightly larger than the water

    - Go to the Initialize Particle Module of the erosion emitter and set the Mesh Scale

        ![](./Anime%20Waterfall/Cylinder02_MeshScale.png)

6. To modify the material's Dynamic Parameters' values, add a Dynamic Material Parameter module to the Particle Spawn

    ![](./Anime%20Waterfall/Cylinder_Parameters.png)

    - Set:
        - TextureTilngX: 1
        - TextureTilingY: 3
        - SpeedY: -0.5

7. Enable the 2nd Dynamic Parameter by Changing "Write Parameter Index 1" to Float and it will show up

    ![](./Anime%20Waterfall/Cylinder02_Parameter2.png)

    - Set:
        - Erosion: 1.3


# Ripples Effect


1. Make a copy of the WaterCylinder01 Emitter node
    - Name it: Ripples01
2. Create a new object using Blender(or other software)
    - Add another Cyclinder
    - In the bottom right menu, set Cap Fill Type: Nothing to remove the top and bottom faces
    - Go to Edit Mode and switch to the Verticies selection mode
    - Press "Alt" + Left Click on one of the top verticies to select everything

        ![](./Anime%20Waterfall/Ripple_Mesh_1.png)

    - Move all the verticies to the center by scaling
        - Press "S" and 0

        ![](./Anime%20Waterfall/Ripple_Mesh_2.png)

    - Move the verticies to the origin
        - Press "G"
        - Press "Z"
        - Press "-" and "1"

    - Select the outer verticies, press "Alt" + "Left Click" on a vertex
    - Move them to the origin
        - Press "G"
        - Press "Z"
        - Press "1"

        ![](./Anime%20Waterfall/Ripple_Mesh_3.png)


    - Switch to Object Mode
    - Select the object, "Right-click" > Shade Smooth to smooth the edges
    - Switch to Edit Mode and add some edge loops
        - Press "Ctrl" + "R"
        - Use the mouse scroll to add 3 loops


        ![](./Anime%20Waterfall/Ripple_Mesh_4.png)


        The Edge loops would make it so that when we use a UV and moe it, the image would expand or retract.

    - Switch back to Object Mode, Press "Ctrl" + "A" and apply "All Transforms"

    - Export as FBX and import to UE


3. Go back to the Niagara System
    - Click on the Mesh Renderer and set the Mesh with the Circle mesh
    - Make sure to the the Erosion material

    ![](./Anime%20Waterfall/Ripples01_1.png)


    
