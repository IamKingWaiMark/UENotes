# Tutorial

https://www.youtube.com/watch?v=nOirmGz8YG4&list=PLRwdP2lOr4A5OA57wRduw_8r06Uvg4EoR

# Steps

1. Create an empty Niagara System and open it
2. Spawn particles
    1. In the event graph, Right-click >  Add Emitter > Minimal
    2. (F2) to rename
    3. Spawn particles, go to "Emitter Update"
        - Add a "Spawn Rate" module
            - Set "Spawn Rate": 1000 (for 1000 particles)
        - Under "Particle Spawn" > "Initialize Particle" (might need to be added), set
            - Lifetime: 3
            - Sprite Size Mode: Uniform
            - Uniform Sprite Size: 3
    4. Define the spawn area, go to "Particle Spawn" and add a "Shape Location" module and set:
        - Shape Priitive: Torus (for a donut)
        - Large Radius: 125
        - Handle Radius: 25
        - Rotation Mode: Axis Angle
    5. Rotate particles, go to "Particle Update" - define what happens to the particles over their lifetime
        - Add a "Vortex Force" module
            - Set:
                - Vortext Axis: 1, 0, 1 (X, Y, Z)
                - Vortext Force Amount : 100
        - Add a "Drag Module"
        - Add a Scale Sprite Size Module (Niagara modifies the size of sprite particles over their lifetime or based on some input value)
            
            ![](./Fluid%20Smoke/Scale%20Sprite%20Size.png)

            - Values means small to large where 0 means nothing and 1 (or more) means max size

            - Set to this:

                ![](./Fluid%20Smoke/Scale%20Sprite%20Size%20Change.png)

                - You need to add a Key Frame in the center (Right-click on the line > Add Key)

                - Press "Ctrl + A" to select all the keys and then "1" to create a smooth curve

                    ![](./Fluid%20Smoke/Scale%20Sprite%20Size%20Change%20Curve.png)

    6. Add Smoke, go "Particle Update" and add a "Set Fluid Source Attribute" module

        - You need to have the NiagaraFluids plugin by Epic Games installled.
        
        ```
        The Set Fluid Source Attribute module in Niagara is used to write particle data into a fluid simulation — specifically marking particles as sources that inject properties into a fluid grid (like smoke, fire, or liquid simulations).
        ```

        - Set:
            - Temperature : 0
            - Density : 0.3
            - Radius: 5
            - Velocity: Scale : 2
                - Changes the dramatic effects of the smoke from the particle

    7. Add a new Emitter for the Smoke
        - Right-click > Add Emitter > Parent Emitters > Grid 3D Gas Master Emitter

        - Get rid of the Sphere Emitter and emit the smoke from the particles
            1. Disable the Sphere Source from the Gas Master Emitter attributes

                ![](./Fluid%20Smoke/DisableSphereSource.png)
            
            2. In the particle emitter (created in Step 1), set 
                - Sim Target : GPU Compute Sim
                    - Moves the particle simulation from the CPU to the GPU
                - Calcuate Bonds Mode: Fix
                    - Instead of dynamically calculating where all particles are every frame, you manually define a fixed bounding box
                    - You set a Min and Max XYZ boundary that permanently wraps the effect

            3. Set:
                - Particle Source Type : Emitter or enable "Use Emitter Source"
                - Emitter Binding > Enter the Name of the Emitter you want to bind to (Step 1)
                
            4. Change the Size and Location of the Bounding box of the Gas Master Emitter so morespace for the smoke.
                - Under Simulation for the Gas Emitter, set
                    - World Size: 800, 1200, 600 (X, Y, Z)
                    - Local Pivot: 0.2, 0, 0.2

            5. Control the Smoke direction
                - Under Simulation, set
                    - Density Buoyancy : 0.1
                        - Negative values means it is going up
                        - Positive means it is going down
                    - Dissipation Rate Density : 0.9
                        - Higher the value the faster it dissipates
                    - Pressure Solve Iterations
                        - Helps collosion with the floor
                    - Draw Bounds: Uncheck (hides the bounding box)
                - Under Forces, set
                    - Wind > Enable Calculate Wind
                        - Wind Magnitude: 10
                            - How strong the wind is
                - Under Collisions
                    - Boundary: Disble Open Boundary-Z 
                        - This will make the smoke collide with the ground instead of dissapearing

            6. Turn off the Sprite Renderer for the particle emitter so only the smoke emitter is shown.

                ![](./Fluid%20Smoke/DisabledRendered.png)


# Extra Particles

1. Copy the Emitter Particle
2. Turn off Set Fluid Source Attribute
3. Enable Sprite Renderer
4. Firefly Effects
    - Spawn Rate: 200
    - Lifetime: 3
    - Uniform Sprite Size: Random Range Float (click on the down arrow to see this option)
        - Minimum: 0.5
        - Maximum: 1.5
5. In Particle Update, add "Curl Noise Force" module and set:
    - Noise Strength: 75
    - This will spread the particles out
6. Update the color, in Initialize Particle, set
    - Color Mode: Random Range
        - Set color Min and Max

        