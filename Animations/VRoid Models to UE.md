# Overview

This documentation shows how to import VRoid models into Unreal Engine and apply Mixamo Animations to them.


# Prerequisite 

- Software:
    - [VRoid Studio](https://vroid.com/en/studio)
    - [Blender](https://www.blender.org/download/)
    - [AccuRIG](https://actorcore.reallusion.com/auto-rig/accurig)
- Blender AddOns
    - [VRM format](https://extensions.blender.org/add-ons/vrm/)


## Installing the VRM format Blender AddOn

1. Go to the [VRM Studio Blender Addon Page](https://extensions.blender.org/add-ons/vrm/) and Click on "Get Add-on" and Drag and Drop the indicator into Blender
    - It might prompt you to "Allow Online Access", go to Edit > Preferences > System > Network > Allow Online Access

## Importing VRoid Model to UE

1. Open VRoid Studio
2. Create Model
3. Export as VRM
4. Open Blender
5. Drag the VRM file into Blender
6. Select the Model (exclude to Amature)
7. Export as FBX
    - Set:
        - Path Mode: Copy
            - Embeded Texture: ✔️
                - This is the icon next to the option
        - Limit to "Selected Objects"
        - Object Types:
            - Mesh
        - Apply Scale: 1.0
        - Apply Transform: ✔️
        - Include > Armatures: ✔️
        - Add Leaf Bones: OFF
        - Apply Unit: ✔️
        - Forward: -Z Forward
        - Up: Y Up (if going to Unreal)
        - Or Z Up (if going to Unity/Misc)
        - Animation: OFF
8. In Blender, delete the model and drag in the exported FBX to check if the materials are there
9. Open AccuRIG
19. Drag the FBX file in to import it
11. Select Rig Body and continue to flow
12. Export the rigged model 
    - You should see a fbx file
13. Open Unreal Engine
14. Drag and Drop the rigged FBX object into the Content Browser and import all objects


## Applying Mixamo Animation to the VRoid Model (Retargeting)

1. Go to [https://www.mixamo.com](https://www.mixamo.com)
2. Download an animation using "With Skin"
3. Import it to UE (all)
4. Open the Skeletal Mesh
    - Click on "Skeleton" tab on the left side
    - Click on "Edit Skeleton"
    - On the Right side, collapse the "Hips" tree 
    - Add a "New Bone" by clicking on the "+" icon
    - Rename the new bone to "Root"
    - Put the "Hips" bone tree inside the "Root" bone
    - Save
5. Download an Animation in Mixamo using "Without Skin" or "No Character" AND "IN PLACE"
6. Import it to UE 
    - Select the modified Maximo Skeletal Mesh as the "Skeleton" and import
7. Right click on the imported Mixamo animation
    - Select:
        1. Retarget Animations
            - For the Source Skeletal Mesh, it should be the modified Skeletal Mesh that was used when importing the animation
            - For the Target, select Skeletal Mesh of the VRoid Model
        2. If there are multiple animations, select all the animation and then click "Export Animations"


## Creating an Animation Blueprint from Scratch

1. Create an Animation Blueprint
    - Select the Rigged Skeleton from the AccuRIG FBX import
2. Go to the Anim Graph
    - Add a "Slot Default Slot" and connect it to the Output Pose
        - This will allow montages to play and overwrite the animations
    - Create State Machines
        - Locomotion
            - Connect this to a Cached Pose so it can be used elsewere in the animation blueprint


