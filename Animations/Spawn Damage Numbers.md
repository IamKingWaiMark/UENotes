# Overview

Spawning damage numbers when hitting enemies


# Setup


1. Create a Widget Blueprint. 
    - This represents the number text that spawns when the enemie is hit

2. Open the Widget Blueprint

3. Add a Canvas and a Text

4. Define a variable call DamageAmount (float / integer) with "Instance Editable" and "Exposed on Spawn"

5. Select the Text component in the "Designer" tab and find the binding section and bind the variable with the Text component

    ![](./Assets/Spawn%20Damage%20Numbers/BindText.png)


6. Create a Actor blueprint for the Damage text

7. Add a Widget Component and set the Widget Class to the Widget you create in Step 1.

    ![](./Assets/Spawn%20Damage%20Numbers/WidgetClass.png)

8. Set "Space" to "Screen" so the widget will always face the player on the screen



# Spawning the Damage Texts

The idea to make this work, is when you "hit" something, it will spawn the actor with the Damage Text in step 6 of [Setup](#setup).



This example, we will use "Line Trace By Channel" for collision detection when an attack occurs and then use the "Spawn Actor of Class" to spawn the damage text.



![](./Assets/Spawn%20Damage%20Numbers/SpawnDamageText.png)



# Adding Animations to the Damage Text


1. Open the Widget Blueprint of the Damage Text in step 1 of [Setup](#setup).

2. In the "Animation" section, add a new Animation by clicking on the "+ Animation" button.

    ![](./Assets/Spawn%20Damage%20Numbers/NewAnimation.png)

3. Expand the timeline that represents the duration of the animation 

    ![](./Assets/Spawn%20Damage%20Numbers/Duration.png)

4. Add the Damage Text component to the Animation by clicking on the "+ Add" button.

    ![](./Assets/Spawn%20Damage%20Numbers/AddDamageText.png)
    
    ![](./Assets/Spawn%20Damage%20Numbers/AddedDamageText.png)

5. Select the Text Component and click on the "+" button to add "tracks" that modifies the attributes of the text component.

    ![](./Assets/Spawn%20Damage%20Numbers/TransformTract.png)

    This example shows that we added the "Transform" track which can be used to modify the position and size.


6. Select the "Transform" track and add a keyframe by clicking on the button with the "+" icon inside a diamond box.

    ![](./Assets/Spawn%20Damage%20Numbers/Timeline%20add%20first%20keyframe.png)

7. Select another point in the timeline and add another keyframe

    ![](./Assets/Spawn%20Damage%20Numbers/Timeline%20second%20keyframe.png)

8. Select the second keyframe and change the attributes of the textbox in the "Details" panel to set the Transform (size / location)

    ![](./Assets/Spawn%20Damage%20Numbers/Timeline%20set%20transform%20attributes.png)


9. To play the animation, open the Damage Actor blueprint you created in step 6 of [Setup](#setup).

    - Get the Widget Component > Get User Widget Object > Cast to the Widget Blueprint > Get the reference of the Animation > Play Animation

    ![](./Assets/Spawn%20Damage%20Numbers/PlayAnimationNodes.png)



# Destorying the Actor when the Animation Finishes

1. Get reference to the Widget or Animation
2. Bind the Animation Finished Event

    ![](./Assets/Spawn%20Damage%20Numbers/BindAnimation.png)

3. Set:
    - Animation = Animation reference from the Widget
    - Target = Widget
    - Delegate
        - Add Event > Add Custom Event

4. Call Desotry Actor when the animation finishes

    ![](./Assets/Spawn%20Damage%20Numbers/DestroyActor.png)