# Straight Line Trace Using Camera's Facing Direction from Socket

This section will show the nodes to shoot a straight line trace from the character's head forward using the camera's rotation.


[Get Player Controller] → [Get Player Camera Manager]
                                      ↓
                              [Get Camera Rotation]
                                      ↓
                              [Break Rotator] → take Yaw only
                                      ↓
                              [Make Rotator] (Pitch: 0, Roll: 0, Yaw: from above)
                                      ↓
                              [Get Forward Vector]
                                      ↓
                           [* 18288] (float multiply)
                                      ↓
[Get Mesh] → [Get Socket Location "head"] → [+] → End
                                             ↑
                                        (add the two together)
                                             
[Get Mesh] → [Get Socket Location "head"] → Start


[Line Trace By Channel]
├── Start: Head Socket Location
├── End:   Head Socket Location + (FlatCameraForward * 18288)
├── Trace Channel: Visibility
└── Draw Debug Type: For Duration


![](./Assets/Useful%20Vector%20Nodes/StraightLineTrace.png)