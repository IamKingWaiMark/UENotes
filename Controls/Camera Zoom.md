# Overview

Smmoth camera zoom effects with Timeline


# Camera Zoom Effect with Timeline

Timeline allows you to transition from one value to another 


1. Add a Timeline

    - Double click to get inside
    - Set the duration to .5 (length in seconds)
    - Add a Float Track
    - Right click into the graph to add 2 Keys
        - Key 1: Time (0) and Value (0)
        - Key 2: Time (.5) and Value (1)
    - Select all the points and right click on one and pick "Auto" to create a curve for a smooth transition effect


    ![](./Assets/Camera%20Zoom/Timeline.png)

    Rename the Track name to "Alpha"

2.  Use the Lerp (Vector) node to blend 2 vector values and set the camera's Spring Arm socket offset

    It takes:

        A = start vector

        B = end vector

        Alpha = how far to move from A toward B

    And returns a new vector between them.

    The idea is:

        Alpha = 0 → returns A

        Alpha = 1 → returns B

        Alpha = 0.5 → returns a point halfway between A and B

    So if:

        A = (0, 0, 0)

        B = (100, 0, 0)

        Alpha = 0.25

        Result = (25, 0, 0)


    ![](./Assets/Camera%20Zoom/Socket%20Offset.png)

# Reversing the Zoom (Zoom out)

When you use a Timeline, you can use the Reverse pin on the Timeline to revert the changes.

This way, you cn zoom in / out.


![](./Assets/Camera%20Zoom/Reverse%20Pin.png)

Alternatively, you can use the Reverse node. 

1. Get a reference to your timeline. Right-click on an empty spot on the blueprint and search for your timeline (name you use when you added it).

![](./Assets/Camera%20Zoom/Timeline%20Ref.png)

![](./Assets/Camera%20Zoom/Timeline%20Ref%20Reverse%20Node.png)