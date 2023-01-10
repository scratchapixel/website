In the first chapter of this lesson, we presented the principle of a pinhole camera. In this chapter, we will show that the size of the photographic film on which the image is projected and the distance between the hole and the back side of the box also play an important role in how a camera delivers images. One possible use of CGI is combining CG images with live-action footage. Therefore, we need our virtual camera to deliver the same type of images as those delivered with a real camera so that images produced by both systems can be composited with each other seamlessly. In this chapter, we will again use the pinhole camera model to study the effect of changing the film size and the distance between the photographic paper and the hole on the image captured by the camera. In the following chapters, we will show how these different controls can be integrated into our virtual camera model.

## Focal Length, Angle Of View, and Field of View

![Figure 1: the sphere projected on the image plane becomes bigger as the image plane moves away from the aperture (or smaller when the image plane gets closer to the aperture). This is equivalent to zooming in and out.](/images/cameras/zoom.png?)

![Figure 2: the focal length is the distance from the hole where light enters the camera to the image plane.](/images/cameras/focallength.png?)

![Figure 3: focal length is one of the parameters that determines the value of the angle of view.](/images/cameras/focallength2.png?)

Similarly to real-world cameras, our camera model will need a mechanism to control how much of the scene we see from a given point of view. Let's get back to our pinhole camera. We will call the back face of the camera the face on which the image of the scene is projected, the **image plane**. Objects get smaller, and a larger portion of the scene is projected on this plane when you move it closer to the aperture: you **zoom out**. Moving the film plane away from the aperture has the opposite effect; a smaller portion of the scene is captured: you **zoom in** (as illustrated in Figure 1). This feature can be described or defined in two ways: distance from the film plane to the aperture (you can change this distance to adjust how much of the scene you see on film). This distance is generally referred to as the **focal length** or **focal distance** (Figure 2). Alternatively, you can also see this effect as varying the angle (making it larger or smaller) of the apex of a triangle defined by the aperture and the film edges (Figures 3 and 4). This angle is called the **angle of view** or **field of view** (or AOV and FOV, respectively).

![Figure 4: the field of view can be defined as the angle of the triangle in the horizontal or vertical plane of the camera. The horizontal field of view varies with the width of the image plane, and the vertical field of view varies with the height of the image plane.](/images/cameras/fov.png?)

![Figure 5: we can use Pythagorean trigonometric identities to find AC if we know both \(\theta\) (which is half the angle of view) and AB (which is the distance from the eye to the canvas).](/images/cameras/angleofview.png?)

In 3D, the triangle defining how much we see of the scene can be expressed by connecting the aperture to the top and bottom edges of the film or to the left and right edges of the film. The first is the **vertical field of view**, and the second is the **horizontal field of view** (Figure 4). Of course, there's no convention here again; each rendering API uses its own. OpenGL, for example, uses a vertical FOV, while the RenderMan Interface and Maya use a horizontal FOV.

As you can see from Figure 3, there is a direct relation between the focal length and the angle of view. So if AB is the distance from the eye to the canvas (so far, we always assumed that this distance was equal to 1, but this won't always be the case, so we need to consider the generic case), AC is half the canvas size (either the width or the height of the canvas), and the angle \(\theta\) is half the angle of view. Because ABC is a right triangle, we can use [Pythagorean trigonometric identities](https://en.wikipedia.org/wiki/Pythagorean_trigonometric_identity) to find AC if we know both \(\theta\) and AB:

$$
\begin{array}{l}
\tan(\theta) = \frac {BC}{AB} \\
BC = \tan(\theta) * AB \\
\text{Canvas Size } = 2 * \tan(\theta) * AB \\
\text{Canvas Size } = 2 * \tan(\theta) * \text{ Distance to Canvas }. 
\end{array}
$$

This is an important relationship because we now have a way of controlling the size of the objects in the camera's view by simply changing one parameter, the angle of view. As we just explained, changing the angle of view can change the extent of a given scene imaged by a camera, an effect more commonly referred to in photography as **zooming in** or **out**.

## Film Size Matters Too

![Figure 6: a larger surface (in blue) captures a larger extent of the scene than a smaller surface (in red). A relation exists between the size of the film and the camera angle of view. The smaller the surface, the smaller the angle of view.](/images/cameras/filmsize3.png?)

![Figure 7: if you use different film sizes but your goal is to capture the same extent of a scene, you need to adjust the focal length (in this figure denoted by A and B).](/images/cameras/filmsize4.png?)

You can see, in Figure 6, that how much of the scene we capture also depends on the film (or sensor) size. In photography, film size or image sensor size matters. A larger surface (in blue) captures a larger extent of the scene than a smaller surface (in red). Thus, a relation also exists between the size of the film and the camera angle of view. The smaller the surface, the smaller the angle of view (Figure 6b).

Be careful. Confusion is sometimes made between film size and image size. There is a relation between the two, of course. The motivation behind developing large formats, whether in film or photography, was mainly image quality. The larger the film, the more details and the better the image quality. However, note that if you use films of different sizes but always want to capture the same extent of a scene, you will need to adjust the focal length accordingly (as shown in Figure 7). That is why a 35mm camera with a 50mm lens doesn't produce the same image as a [large format](https://en.wikipedia.org/wiki/Large_format_(photography)) camera with a 50mm lens (in which the film size is about at least three times larger than a 35mm film). The focal length in both cases is the same, but because the film size is different, the angular extent of the scene imaged by the large format camera will be bigger than that of the 35mm camera. It is crucial to remember that the size of the surface capturing the image (whether in digital or film) also determines the angle of view (as well as the focal length).

<details>
The terms **film back** or **film gate** technically designate two things slightly different, but they both relate to film size, which is why the terms are used interchangeably. The first term relates to the film holder, a device generally placed at the back of the camera to hold the film. The second term designates a rectangular opening placed in front of the film. By changing the gate size, we can change the area of the 35 mm film exposed to light. This allows us to change the film format without changing the camera or the film. For example, CinemaScope and Widescreen are formats shot on 35mm 4-perf film with a film gate. Note that film gates are also used with digital film cameras. The film gate defines the film aspect ratio.

![](/images/cameras/filmgate.png?)

The 3D application Maya groups all these parameters in a Film Back section. For example, when you change the Film Gate parameter, which can be any predefined film format such as 35mm Academy (the most common format used in film) or any custom format, it will change the value of a parameter called Camera Aperture, which defines the horizontal and vertical dimension (in inch or mm) of the film. Under the Camera Aperture parameter, you can see the Film Aspect Ratio, which is the ratio between the "physical" width of the film and its height. See [list of film formats](https://en.wikipedia.org/wiki/List_of_film_formats) for a table of available formats.

![](/images/cameras/mayacamera.png?)

At the end of this chapter, we will discuss the relationship between the film aspect ratio and the image aspect ratio.

</details>

!!!
It is important to remember that two parameters determine the angle of view: the focal length and the film size. The angle of view changes when you change either one of these parameters: the focal length or the film size.

- For a fixed film size, changing the focal length will change the angle of view. The longer the focal length, the narrower the angle of view.
- For a fixed focal length, changing the film size will change the angle of view. The larger the film, the wider the angle of view.
- If you wish to change the film size but keep the same angle of view, you will need to adjust the focal length accordingly.
  !!!

![Figure 8: 70 mm (left) and 24x35 film (right).](/images/cameras/filmformat.png?)

Note that three parameters are inter-connected, the angle of view, the focal length, and the film size. With two parameters, we can always infer the third one. Knowing the focal length and the film size, you can calculate the angle of view. If you know the angle of view and the film size, you can calculate the focal length. The next chapter will provide the mathematical equations and code to calculate these values. Though in the end, note that we want the angle of view. If you don't want to bother with the code and the equations to calculate the angle of view from the film size and the focal length, you don't need to do so; you can directly provide your program with a value for the angle of view instead. However, in this lesson, our goal is to simulate a real physical camera. Thus, our model will effectively take into account both parameters.

<details>
The choice of a film format is generally a compromise between cost, the workability of the camera (the larger the film, the bigger the camera), and the image definition you need. The most common film format (known as the [135 camera film format](https://en.wikipedia.org/wiki/135_film)) used for still photography was (and still is) 36 mm (1.4 in) wide (this file format is better known for being 24 by 35 mm however the exact horizontal size of the image is 36 mm). The next larger size of film for still cameras is the medium format film which is larger than 35 mm (generally 6 by 7 cm), and the large format, which refers to any imaging format of 4 by 5 inches or larger. Film formats used in filmmaking also come in a large variety of sizes. Refrain from assuming that because we now (mainly) use digital cameras, we should not be concerned by the size of the film anymore. Rather than the size of the film, it is the size of the sensor that we will be concerned about for digital cameras, and similarly to film, that size also defines the extent of the scene being captured. Not surprisingly, sensors you can find on high-end digital DLSR cameras (such as the Canon 1D or 5D) have the same size as the 135 film format: they are 36 mm wide and have a height of 24 mm (Figure 8).
</details>

## Image Resolution and Frame Aspect Ratio

**The size of a film (measured in inches or millimeters) is not to be confused with the number of pixels in a digital image**. The film's size affects the angle of view, but the image resolution (as in the number of pixels in an image) doesn't. These two camera properties (how big is the image sensor and how many pixels fit on it) are independent of each other.

![Figure 9: image sensor from a Leica camera. Its dimensions are 36 by 24 mm. Its resolution is 6000 by 4000 pixels.](/images/cameras/sensor.png?)

![Figure 10: some common image aspect ratios (the first two examples were common in the 1990s. Today, most cameras or display systems support 2K or 4K image resolutions).](/images/cameras/imageaspectratio.png?)

In digital cameras, the film is replaced by a sensor. An image sensor is a device that captures light and converts it into an image. You can think of the sensor as the electronic equivalent of film. The image quality depends not only on the size of the sensor but also on how many millions of pixels fit on it. It is important to understand that the film size is equivalent to the sensor size and that it plays the same role in defining the angle of view (Figure 9). However, the number of pixels fitting on the sensor, which defines the image resolution, has no effect on the angle and is a concept purely specific to digital cameras. Pixel resolution (how many pixels fit on the sensor) only determines how good images look and nothing else.

The same concept applies to CG images. We can calculate the same image with different image resolutions. These images will look the same (assuming a constant ratio of width to height), but those rendered using higher resolutions will have more detail than those rendered at lower resolutions. The resolution of the frame is expressed in terms of pixels. We will use the terms **width** and **height resolution** to denote the number of pixels our digital image will have along the horizontal and vertical dimensions. The image itself can be seen as a gate (both the image and the film gate define a rectangle), and for this reason, it is referred to in Maya as the **resolution gate**. At the end of this chapter, we will study what happens when the resolution and film gate **relative size** don't match.

One particular value we can calculate from the image resolution is the **image aspect ratio**, called in CG the **device aspect ratio**. Image aspect ratio is measured as:

$$\text{Image (or Device) Aspect Ratio} = { width \over height }$$

When the width resolution is greater than the height resolution, the image aspect ratio is greater than 1 (and lower than 1 in the opposite case). This value is important in the real world as most films or display devices, such as computer screens or televisions, have standard aspect ratios. The most common aspect ratios are:

- **4:3**. It was the aspect ratio of old television systems and computer monitors until about 2003; It is still often the default setting on digital cameras. While it seems like an old aspect ratio, this might be true for television screens and monitors, but this is not true for film. The 35mm film format has an aspect ratio of 4:3 (the dimension of one frame is 0.980x0.735 inches).
- **5:3** and **1.85:1**. These are two very common standard image ratios used in film.
- **16:9**. It is the standard image ratio used by high-definition television, monitors, and laptops today (with a resolution of 1920x1080).

<details>
The RenderMan Interface specifications set the default image resolution to 640 by 480 pixels, giving a 4:3 Image aspect ratio.
</details>

## Canvas Size and Image Resolution: Mind the Aspect Ratio!

Digital images have a particularity that physical film doesn't have. The aspect ratio of the sensor or the aspect ratio of what we called the canvas in the previous lesson (the 2D surface on which the image of a 3D scene is drawn) can be different from the aspect ratio of the digital image. You might think: "why would we ever want that anyway?". Generally, indeed, this is something other than what we want, and we are going to show why. And yet it happens more often than not. Film frames are often scanned with a gate different than the gate they were shot with, and this situation also arises when working with [anamorphic formats](https://en.wikipedia.org/wiki/Anamorphic_format) (we will explain what anamorphic formats are later in this chapter).

![Figure 11: if the image aspect ratio is different than the film size or film gate aspect ratio, the final image will be stretched in either x or y.](/images/cameras/aspectratio.png?)

Before we consider the case of anamorphic format, let's first consider what happens when the canvas aspect ratio is different from the image or device aspect ratio. Let's take a simple example: what we called the canvas in the previous lesson is a square, and the image on the canvas is that of a circle. We will also assume that the coordinates of the lower-left and upper-right corners of the canvas are [-1,1] and [1,1], respectively. Recall that the process for converting pixel coordinates from screen space to raster space consists of first converting the pixel coordinates from screen space to NDC space and then NDC space to raster space. In this process, the NDC space is the space in which the canvas is remapped to a unit square. From there, this unit square is remapped to the final raster image space. Remapping our canvas from the range [-1,1] to the range [0,1] in x and y is simple enough. Note that both the canvas and the NDC "screen" are square (their aspect ratio is 1:1). Because the "image aspect ratio" is preserved in the conversion, the image is not stretched in either x or y (it's only squeezed down within a smaller "surface"). In other words, visually, it means that if we were to look at the image in NDC space, our circle would still look like a circle. Let's imagine now that the final image resolution in pixels is 640x480. What happens now? The image, which originally had a 1:1 aspect ratio in screen space, is now remapped to a raster image with a 4:3 ratio. Our circle will be stretched along the x-axis, looking more like an oval than a circle (as depicted in Figure 11). **Not preserving the canvas aspect ratio and the raster image aspect ratio leads to stretching the image in either x or y**. It doesn't matter if the NDC space aspect ratio is different from the screen and raster image aspect ratio. You can very well remap a rectangle to a square and then a square back to a rectangle. All that matters is that both rectangles have the same aspect ratio (obviously, stretching is something we want only if the effect is desired, as in the case of anamorphic format).

You may think again, "why would that ever happen anyway?". Generally, it doesn't happen because, as we will see in the next chapter, the canvas aspect ratio is often directly computed from the image aspect ratio. Thus if your image resolution is 640x480, we will set the canvas aspect ratio to 4:3.

![Figure 12: when the resolution and film gates are different (top), you need to choose between two possible options. You can either fit the resolution gate within the film gate (middle) or the film gate within the resolution gate (bottom). Note that the renders look different.](/images/cameras/filmgate3.png?)

However, you may calculate the canvas aspect ratio from the film size (called Film Aperture in Maya) rather than the image size and render the image with a resolution whose aspect ratio is different than that of the canvas. For example, the dimension of a 35mm film format (also known as academy) is 22mm in width and 16mm in height (these numbers are generally given in inches), and the ratio of this format is 1.375. However, a standard 2K scan of a full 35 mm film frame is 2048x1556 pixels, giving a device aspect ratio of 1.31. Thus, the canvas and the device aspect ratios are not the same in this case! What happens, then? Software like Maya offers different user strategies to solve this problem. No matter what, Maya will force at render time your canvas ratio to be the same as your device aspect ratio; however, this can be done in several ways:

- You can either force the resolution gate within the film gate. This is known as the **Fill** mode in Maya.
- Or you can force the film gate within the resolution gate. This is known as the **Overscan** mode in Maya.

Both modes are illustrated in Figure 12. Note that if the resolution gate and the film gate are the same, switching between those modes has no effect. However, when they are different, objects in the overscan mode appear smaller than in the fill mode. We will implement this feature in our program (see the last two chapters of this lesson for more detail).

![](/images/cameras/filmgate2.png?)

<details>
What do we do in film production? The Kodak standard for scanning a frame from a 35mm film in 2K is 2048x1556, The resulting 1.31 aspect ratio is slightly lower than the actual film aspect ratio of a full aperture 35mm film, which is 1.33 (the dimension of the frame is 0.980x0.735 inches). This means that we scan slightly more of the film than what's strictly necessary for height (as shown in the adjacent image). Thus, if you set your camera aperture to "35mm Full Aperture", but render your CG renders at resolution 2048x1556 to match the resolution of your 2K scans, the resolution and film aspect ratio won't match. In this case, because the actual film gate fits within the resolution gate during the scanning process, you need to select the "Overscan" mode to render your CG images. This means you will render slightly more than you need at the frame's top and bottom. Once your CG images are rendered, you will be able to composite them to your 2K scan. But you will need to crop your composited images to 2048x1536 to get back to a 1.33 aspect ratio if required (to match the 35mm Full Aperture ratio). Another solution is scanning your 2K images to exactly 2048x1536 (1.33 aspect ratio), another common choice. That way, both the film gate and the resolution gate match.
</details>

</details>
The only exception to keeping the canvas and the image aspect ratio the same is when you work with **anamorphic formats**. The concept is simple. Traditional 35mm film cameras have a 1.375:1 gate ratio. To shoot with a widescreen ratio, you need to put a gate in front of the film (as shown in the adjacent image). What it means, though, is that part of the film is wasted. However, you can use a special lens called an anamorphic lens, which will compress the image horizontally so that it fits within as much of the 1.375:1 gate ratio as possible. When the film is projected, another lens stretches images back to their original proportions. The main benefit of shooting anamorphic is the increased resolution (since the image uses a larger portion of the film). Typically anamorphic lenses squeeze the image by a factor of two. For instance, Star Wars (1977) was filmed in a 2.35:1 ratio using an anamorphic camera lens. If you were to composite CG renders into Star Wars footage, you would need to set the resolution gate aspect ratio to ~4:3 (the lens squeezes the image by a factor of 2; if the image ratio is 2:35, then the film ratio is closer to 1.175), and the "film" aspect ratio (the canvas aspect ratio) to 2.35:1. In CG this is typically done by changing what we call the pixel aspect ratio. In Maya, there is also a parameter in the camera controls called Lens Squeeze Ratio, which has the same effect. But this is left to another lesson.

![](/images/cameras/anamorphic.png?)

</details>

## Conclusion and Summary: Everything You Need to Know about Cameras

What is important to remember from the last chapter is that all that matters at the end is the camera's angle of view. You can set its value directly to get the visual result you want.

> I want to combine real film footage with CG elements. The real footage is shot and loaded into Maya as an image plane. Now I want to set up the camera (manually) and create some rough 3D surroundings. I noted down a couple of camera parameters during the shooting and tried to feed them into Maya, but it didn't work out. For example, if I enter the focal length, the resulting view field is too big. I need to familiarize myself with the relationship between focal length, film gate, field of view, etc. How do you tune a camera in Maya to match a real camera? How should I tune a camera to match these settings?

However, Suppose you wish to build a camera model to simulate physical cameras (the goal of the person we quoted above). In that case, you will need to compute the angle of view by considering the focal length and the film gate size. Many applications, such as Maya, expose these controls (the image below is a screenshot of Maya's UI showing the Render Settings and the Camera attributes). You now understand exactly why they are there, what they do and how to set their value to match the result produced by a real camera. If your goal is to combine CG images with live-action footage, you will need to know the following:

- The film gate size. This information is generally given in inches or mm. This information is always available in camera specifications.

- The focal length. Remember that the angle of view depends on film size for a given focal length. In other words, if you set the focal length to a given value but change the film aperture, the object size will change in the camera's view.

![](/images/cameras/mayacamcontrols.png?)

However, remember that the resolution gate ratio may differ from the film gate ratio, which you only want if you work with anamorphic formats. For example, suppose the resolution gate ratio of your scan is smaller than the film gate ratio. In that case, you will need to set the Fit Resolution Gate parameter to Overscan as with the example of 2K scans of 35mm full aperture film, whose ratio (1.316:1) is smaller than the actual frame ratio (1.375:1). You need to pay a great deal of attention to this detail if you want CG renders to match the footage.

Finally, the only time when the "film gate ratio" can be different from the "resolution gate ratio" is when you work with anamorphic formats (which is quite rare, though).

## What's Next?

We are now ready to develop a virtual camera model capable of producing images that match the output of real-world pinhole cameras. In the next chapter, we will show that the angle of view is the only thing we need if we use ray tracing. However, if we use the rasterization algorithm, we must compute the angle of view and the canvas size. We will explain why we need these values in the next chapter and how we can compute them in chapter four.