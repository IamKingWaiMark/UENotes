# Overview

In Unreal Engine, changing color effects based on location is done through a system called Post Process Volumes. 


# Post Process Volumes

1. Place the volume — In the editor, go to Place Actors panel, search for "Post Process Volume", and drag it into your level. Scale it to cover the area you want.

2. Configure the effects — Select the volume and open the Details panel. You'll find settings organized into sections:

    - Color Grading — Saturation, contrast, gamma, gain, lift (split-toning), white balance
    - Bloom — Glow intensity around bright lights
    - Lens Flares — Atmospheric lens effects
    - Ambient Occlusion — How dark crevices appear
    - Exposure — Auto-exposure min/max, compensation
    - Chromatic Aberration — Color fringing on screen edges
    - Vignette — Darkening around screen edges

3. Set the blend radius — The Blend Radius property controls how gradually the effect fades in as the camera approaches and enters the volume. A value of 0 means instant snap; 500–1000 gives a smooth transition.

4. Unbound option — Check Infinite Extent (Unbound) if you want the settings to apply globally across the entire level, not just inside a box.


- Multiple zones

    You can stack as many Post Process Volumes as you want. Each gets a Priority number — higher priority wins when volumes overlap. This lets you create distinct looks per zone (a warm cave vs. a cold mountain peak, for example).

- Blending via Blueprint

    If you need dynamic control (like fading effects in response to gameplay events), you can adjust Post Process Volume properties at runtime using Blueprints with the Set Scalar Parameter Value or by toggling the volume's Blend Weight to smoothly interpolate the effect in and out.

- Using a Post Process Material

    For fully custom effects (heat distortion, night vision, custom color LUTs), you can create a Material with the Post Process domain and assign it to the volume's Post Process Materials array slot.