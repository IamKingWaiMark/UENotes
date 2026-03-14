# Retrieving the Controlled Character

When the Animation Blueprint is applied to the character blueprint, it will automatically set the Pawn Owner context.

So we can use the "Try Get Pawn Owner" node followed by a "Cast to Character" node to get the Character object.


![](./Assets/Useful%20Animation%20Blueprint%20Nodes/Get%20Character.png)

# Getting the Forward Moving Speed

From the [Character](#retrieving-the-controlled-character), get the "Character Movement", then get the "Velocity", followed by "Vectory Length XY".

This will get the character forward moving speed.

![](./Assets/Useful%20Animation%20Blueprint%20Nodes/Getting%20Forward%20Moving%20Speed.png)


# Get Character facing direction

![](./Assets/Useful%20Animation%20Blueprint%20Nodes/Get%20Character%20Facing%20Direction.png)

The Clamping depends soley on your Blend Space needs. In this example, it uses the default UE Walk/Run animation which extends the direction between -180 and 180.

When Orient Rotaation To Movement is true, this means that the character will rotate based on the movement commands so we do not want to play the backwards animation. 

It should never play it because if the character is currently facing forward and you press the back button, the character would turn to that direction so it will continue to play the forward animation.

When Orient Rotation To Movement is false, it should play the backwards animtion since it can start moving backwards. In the Default blend space, the backward animations are in the -180 and 180 sides.


# Check if character is falling

![](./Assets/Useful%20Animation%20Blueprint%20Nodes/Check%20if%20Character%20is%20Falling.png)