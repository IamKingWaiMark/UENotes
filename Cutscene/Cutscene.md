# Steps

1. Create a Level Sequence
2. Add two Camera Actor / Cine Camera Actor to the scene
3. Select one of them and click on the + Track button in the level sequence
4. Select Add Actor Track > Add CameraActor
5. Do it to the other camera as well
6. You can move the camera with key frames using the Transform track for the camera
    

# Camera Rig Rail

1. Add a Camera Rig Rail into the sceene
2. Move the Camera actor into the Rig Rail like a child in the Outliner
3. Set the Location to 0,0,0 for the camera in the rig rail
4. Extend the track by selecting the Rig Rail and movingthe spline points or add more
5. Add the Rig Rail into the Sequencer
6. Add a Current Position on Rail track for the Rig Rail by clickig on the "+" button
    - 0 is the start of the track and 1 is the end

    To make the camera rotate with the rail, click on the Rig Rail track and in the Details, enable "Lock Orientation to Rail

# Spawn Characters

1. Add the character to the level
2. Add it to the sequence
3. Convert the actor to Spawnable
    - Right click on the character track > Convert to Spawnable
    - Delete the original character from the level
    - Add key frames for when to spawn the character and the transforms
4. Apply animations to the character
    - Click the "+" button in the character track and go to Animation > Select an animation