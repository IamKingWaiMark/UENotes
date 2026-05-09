# Overview

Using the Engine's SkySphere blueprint.


# Morning and Night Effects

1. Make a copy of these files and rename them
    - BP_Sky_Sphere
    - M_Sky_Panning_Cloud2
    - SM_SkySphere
2. Add a BP_Sky_Sphere to your level
3. For:
    - Morning, set the "Sun Height" from 0-2
    - Night, set the "Sun Height" from -0.99 - -1

# Using a Custom Star Map

1. Create a Star Map
2. Open the copy of M_Sky_Panning_Cloud2 and look for the "SkyColors" section (commented)
3. Replace the Texture Sample with the new Star Map
    - Set UVs to 2 so it would take the half circle
4. Open the BP_Sky_Sphere and navigate to the Costructor
5. For the "Create Dynamic Material Instance node, replace the "Source Material" with the copy of the M_Sky_Panning_Cloud2 you did in Step 1 of [Morning and Night Effects](#morning-and-night-effects)

