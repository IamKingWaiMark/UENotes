# Tutorial

https://www.youtube.com/watch?v=QmsaO_EWrxA&list=PLhlDHJpnPTG-WUD_dqu0elvgbbhxklWIu&index=2

# 1 First Layer

1. Create a Material
2. Enable "Use Material Attributes" from the material node
    - Sampler Source: Shared Wrap
3. Add a MakeMaterialAttributes node
4. Create connection

    - Texture Sample Texture: DefaultWhiteGrid

    ![](./Assets/1.png)

5. Control the Tiling via Landscape Texture Coords
    - Add a LandscapeLayerCoords node and connect it to eh UVs pi for the Texture Sample

        - To manipulate the tiling, there needs to be some manipulation between the LandscapeLayerCoords node and the UVs pin

        ![](./Assets/2.png)

6. Create a Material Instance of the Material and Swap it in the Landscape Texture

7. Using Normals,
    - In the Material, duplicate the Texture Sample and set the new texture to Grid_N

    ![](./Assets/3.png)

8. Modifying the tint, color of the material

    ![](./Assets/4.png)

9. Altering the Normal (sharpness / prenounce)

    ![](./Assets/5.png)

10. Add the "Forest Path" material from FAB

11. Parametertize the Texture Samples and update the samples in the material instance.



# 2 Distance Blender (CameraDepthFade)

1. Create name rereoute nodes for the new Parameters 

![](./Assets/6.png)

2. Connect it 

![](./Assets/7.png)

3. Make the texture samples for the far UV for the base and normal

![](./Assets/8.png)

![](./Assets/9.png)


4. Use a Mask to interchange between far and near

    - Add a CameraDepthFade
    - Create parameters:
        - Blend Distance Transition
        - Blend Distance Start

    ![](./Assets/10.png)

5. Use the blend for base material and normal

![](./Assets/11.png)

Results

![](./Assets/12.png)


# 3 Slope Mask - Multilayer Textures

1. Get more landscape textures from FAB
    - Mossy Grass
    - Rock Cliff
    - Stony Soil

2. Create a TextureObjectParameter and set it to the WhiteGrid texture. Then apply it to the texture samples Tex pin.

    - Source Sampler: Shared Wrap

    ![](./Assets/13.png)

3. Do the same to the normals.

    - Source Sampler: Shared Wrap
    - Sampler Type: Normal

4. Make everything into a function

    - Select everything for the landscape layer except for the parameters

        ![](./Assets/14.png)

5. Right click on any of the selected nodes and select > "Collapse to Function"
    - Name: MF_CreateLayer

6. Fix the Function. Open it.

    - Select the UV inputs and set the InputType to Vector2
    - Select the TextureObjectParameters and set InputType to Texture2D
    - Rename nodes if needed
    - Set the Sort Priority of the input nodes in the function so we can change the order of the pins in the Material Editor

7. Duplicate the Ground layer stuff and rename everything with "MidLow" including the group names

    ![](./Assets/16.png)

8. Connect the output of the MidLow layer to the final output and apply the "Mossy Grass" texture 

    ![](./Assets/17.png)


9. Do the samefor MidHigh - use Grassy Soil

10. Repeat for Cliff - Use Rock Cliff

11. Add a Slope Mask for the layers
    - FalloffPower 
        - 0, appear everywhere in the landscape


    ![](./Assets/18.png)

12. Blend the materials with the Slope Masks

    Example with 2 materials

    ![](./Assets/19.png)

    All

    ![](./Assets/20.png)


# 4 Snow - Height Mask


https://www.youtube.com/watch?v=lejJ88PZkT8&list=PLhlDHJpnPTG-WUD_dqu0elvgbbhxklWIu&index=5

Important:

For the Texture Sample node, set Sampler Type to Linear Color

Also make sure the noise texture has sRGB turned off


1. Add the "Fresh Windswepy Snow" texture from FAB

2. Create a new Layer section for the Snow

3. Create a Snow Slope Mask

    ![](./Assets/21.png)


4. Blend it for testing

    ![](./Assets/22.png)

5. Use a Height Mask to only display the snow in a certain height

    - Collapse the Snow Slop Mask stuff into function

        ![](./Assets/23.png)

        Function

        ![](./Assets/24.png)

    - Create the Height Mask and combine the slope and height masks together by multiplying

        ![](./Assets/25.png)

6. Add Noise Mask

    ![](./Assets/26.png)

7. Add Switch parameter to apply snow or not

    - Use the Static Snow Parameter

    ![](./Assets/27.png)


# 5 COLOR VARIATION

https://www.youtube.com/watch?v=An2J0dbbg8g&list=PLhlDHJpnPTG-WUD_dqu0elvgbbhxklWIu&index=6


1. Create a new Material Function 

    - MF_ColorVariation


    ![](./Assets/28.png)

    Use it in the master material

    ![](./Assets/29.png)


    - Reults

    ![](./Assets/30.png)

2. Add Intensity

    ![](./Assets/31.png)

3. Add Constrast (Harder or lower blend)

    ![](./Assets/32.png)

4. Add a StaticBoolParameter to turn on/off the variation in the master material

    ![](./Assets/33.png)


# 6 Tiling Variations

1. Create a new Material Function

    - MF_TilingVariation
    
    ![](./Assets/34.png)

2. Use it in MF_CreateLayer

    ![](./Assets/35.png)


# 7 PUDDLES

https://www.youtube.com/watch?v=SfkrZEgaKLw&list=PLhlDHJpnPTG-WUD_dqu0elvgbbhxklWIu&index=8


1. Create a new Material Function

    - MF_Puddles


    ![](./Assets/36.png)


# 8 TRIPLANAR PROJECTION

https://www.youtube.com/watch?v=DUDfnI2ZNuI&list=PLhlDHJpnPTG-WUD_dqu0elvgbbhxklWIu&index=9

![](./Assets/37.png)

1. Use in Create Layer

![](./Assets/39.png)

![](./Assets/40.png)

![](./Assets/41.png)


# 9 Foilage - Landscape Grass Types

https://www.youtube.com/watch?v=szkdAc8X8fY&list=PLhlDHJpnPTG-WUD_dqu0elvgbbhxklWIu&index=10



1. Create a Landscape Grass Type for each Layer

    - Right Click > Foilage > Landscape Grass Type

        - GT_Snow
        - GT_Cliff
        - GT_MidHigh
        - GT_MidLow
        - GT_Ground

2. Open the Master Material

    - Add a Landscape Grass Output node
        - Add the Landscape Grass Types

        ![](./Assets/42.png)


    - Connect the layer masks to the node

        - The subtraction is used to remove the masks from the previous levels so they do not overlap

        ![](./Assets/43.png)

3. To test each layer, you plug in each layer into the final output temporarily

    ![](./Assets/44.png)

    Results

    ![](./Assets/45.png)


4. Download some foilage

    - Real Stones Pack 01

5. Open the Landscape Grass Type file for Ground
    - GT_Ground

        - Add the stone model

        ![](./Assets/46.png)