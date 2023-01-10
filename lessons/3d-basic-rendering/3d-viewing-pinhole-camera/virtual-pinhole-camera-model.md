Our next step is to develop a virtual camera working on the same principle as a pinhole camera. More precisely, our goal is to create a camera model delivering images similar to those produced by a real pinhole camera. For example, if we take a picture of a given object with a pinhole camera, then when a 3D replica of that object is rendered with our virtual camera, the size and shape of the object in the CG render must match exactly the size and shape of the real object in the photograph. But before we start looking into the model itself, it is important to learn a few more things about computer graphics camera models.

First, the details:

- CG cameras have a near and far clipping plane. Objects closer than the near-clipping plane or farther than the far-clipping plane are invisible to the camera. This lets us can exclude some of a scene's geometry and render only certain portions of the scene. This is necessary for rasterization to work.
- In this chapter, we will also see why in CG, the image plane is positioned in front of the camera's aperture rather than behind, as with real pinhole cameras. This plays an important role in how cameras are conventionally defined in CG.
- Finally, we must look into how we can render a scene from any given viewpoint. We discussed this in the previous lesson, but this point will be briefly covered in this chapter.

The important question we haven't looked into yet (asked and answered) is, "studying real cameras to understand how they work is great, but how is the camera model being used to produce images?". We will show in this chapter that the answer to this question depends on whether we use the rasterization or ray-tracing rendering technique.

In this chapter, we will first review the points listed above one by one to give a complete "picture" of how cameras work in CG. Then, the virtual camera model will be introduced and implemented in a program in this lesson's next (and final) chapter.

## How Do We Represent Cameras in the CG World?

Photographs produced by real-world pinhole cameras are upside down. This is happening because, as explained in the first chapter, the film plane is located behind the center of the projection. However, this can be avoided if the projection plane lies on the same side as the scene, as shown in Figure 1. In the real world, the image plane can't be located in front of the aperture because it will not be possible to isolate it from unwanted light, but in the virtual world of computers, constructing our camera that way is not a problem. Conceptually, by construction, this leads to seeing the hole of the camera (which is also the center of projection) as the actual position of the eye, and the image plane, the image that the eye is looking at.

![Figure 1: for our virtual camera, we can move the image plane in front of the aperture. That way, the projected image of the scene on the image plane is not inverted.](/images/cameras/pinholecam1.png?)

Defining our virtual camera that way shows us more clearly how constructing an image by following light rays from wherever point in the scene they are emitted from to the eye turns out to be a simple geometrical problem which was given the name of (as you know it now) **perspective projection**. Perspective projection is a method for building an image through this apparatus, a sort of pyramid whose apex is aligned with the eye, whose base defines the surface of a canvas on which the image of the 3D scene is "projected" onto.

## Near and Far Clipping Planes and the Viewing Frustum

The **near** and **far clipping** planes are virtual planes located in front of the camera and parallel to the image plane (the plane on which the image is contained). The location of each clipping plane is measured along the camera's line of sight (the camera's local z-axis). They are used in most virtual camera models and have no equivalent in the real world. Objects closer than the near-clipping plane or farther than the far-clipping plane are invisible to the camera. Scanline renderers using the z-buffer algorithm, such as OpenGL, need these clipping planes to control the range of depth values over which the objects' depth coordinates are remapped when points from the scene are projected onto the image plane (and this is their primary if only function). Without getting into too many details, adjusting the near and far clipping planes can also help resolve precision issues with this type of renderer. The next lesson will find more information on this problem known as z-fighting. In ray tracing, clipping planes are not required by the algorithm to work and are generally not used.

![Figure 2: any object contained within the viewing frustum is visible.](/images/cameras/frustum.png?)

## The Near Clipping Plane and the Image Plane

![Figure 3: the canvas can be positioned anywhere along the local camera z-axis. Note that its size varies with position.](/images/cameras/clippingplanescanvas1.png?)

![Figure 4: The canvas is positioned at the near-clipping plane in this example. The bottom-left and top-right coordinates of the canvas are used to determine whether a point projected on the canvas is visible to the camera.](/images/cameras/canvascoordinates5.png?)

The canvas (also called screen in other CG books) is the 2D surface (a bounded region of the image plane) onto which the scene's image is projected. In the previous lesson, we placed the canvas 1 unit away from the eye by convention. However, the position of the canvas along the camera's local z-axis doesn't matter. We only made that choice because it simplified the equations for computing the point's projected coordinates, but, as you can see in Figure 3, the projection of the geometry onto the canvas produces the same image regardless of its position. Thus you are not required to keep the distance from the eye to the canvas equal to 1. We also know that the viewing frustum is a truncated pyramid (the pyramid's base is defined by the far clipping plane, and the top is cut off by the near clipping plane). This volume defines the part of the scene that is visible to the camera. A common way of projecting points onto the canvas in CG is to remap points within the volume defined by the viewing frustum to the **unit cube** (a cube of side length 1). This technique is central to developing the perspective projection matrix, which is the topic of our next lesson. Therefore, we don't need to understand it for now. What is interesting to know about the perspective projection matrix in the context of this lesson, though, is that it works because the image plane is located near the clipping plane. We won't be using the matrix in this lesson nor studying it; however, in anticipation of the next lesson devoted to this topic, we will place the canvas at the near-clipping plane. Remember that this is an arbitrary decision and that unless you use a special technique, such as the perspective projection matrix that requires the canvas to be positioned at a specific location, it can be positioned anywhere along the camera's local z-axis.

From now on, and for the rest of this lesson, we will assume that the canvas (or screen or image plane) is positioned at the near-clipping plane. Remember that this is just an arbitrary decision and that the equations we will develop in the next chapter to project points onto the canvas work independently from its position along the camera's line of sight (which is also the camera z-axis). This setup is illustrated in Figure 4.

Remember that the distance between the eye and the canvas, the near-clipping plane, and the focal length are also different things. We will focus on this point more fully in the next chapter.

## Computing the Canvas Size and the Canvas Coordinates

![Figure 5: side view of our camera setup. Objects closer than the near-clipping plane or farther than the far-clipping plane are invisible to the camera. The distance from the eye to the canvas is defined as the near-clipping plane. The canvas size depends on this distance (Znear) and the angle of view. A point is only visible to the camera if the projected point's x and y coordinates are contained within the canvas boundaries (in this example, P1 is visible because P1' is contained within the limits of the canvas, while P2 is invisible).](/images/cameras/clippingplanes2.png?)

![Figure 6: a point is only visible to the camera if the projected point x- and y-coordinates are contained within the canvas boundaries (in this example P1 is visible because P1' is contained within the limits of the canvas, while P2 is invisible).](/images/cameras/clippingplanes1.png?)

![Figure 7: the canvas coordinates are used to determine whether a point lying on the image plane is visible to the camera.](/images/cameras/canvascoordinates6.png?)

We insisted a lot in the previous section on the fact that the canvas could be anywhere along the camera's local z-axis because that position affects the canvas size. When the distance between the eye and the canvas decreases, the canvas gets smaller, and when that distance increases, it gets bigger. The bottom-left and top-right coordinates of the canvas are directly linked to the canvas size. Once we know the size, computing these coordinates is trivial, considering that the canvas (or screen) is centered on the origin of the image plane coordinate system. Why are these coordinates important? Because they can be used to easily check whether a point projected on the image plane lies within the canvas and is, therefore, visible to the camera. Two points are projected onto the canvas in figures 5, 6, and 7. One of them (P1') is within the canvas limits and visible to the camera. The other (P2') is outside the boundaries and is thus invisible. When we know both the canvas coordinates and the projected coordinates, testing if the point is visible is simple.

Let's see how we can mathematically compute these coordinates. In the second chapter of this lesson, we gave the equation to compute the canvas size (we will assume that the canvas is a square for now, as in figures 3, 4, and 6):

$$\text{Canvas Size} = 2 * \tan({\theta \over 2}) * \text{Distance to Canvas}$$

**Where \(\theta\) is the angle of view** (hence the division by 2). Note that the vertical and horizontal angles of view are the same when the canvas is a square. Since the distance from the eye to the canvas is defined as the near clipping plane, we can write:

$$\text{Canvas Size} = 2 * \tan({\theta \over 2}) * Z_{near}.$$

Where \(Z_{near}\) is the distance between the eye and the near-clipping plane along the camera's local z-axis (Figure 5), since the canvas is centered on the image plane coordinate system's origin, computing the canvas's corner coordinates is trivial. But first, we need to divide the canvas size by 2 and set the sign of the coordinate based on the corner's position relative to the coordinate system's origin:

$$
\begin{array}{l}
\text{top} &=&&\dfrac{\text {canvas size}}{2}\\
\text{right} &=&&\dfrac{\text {canvas size}}{2}\\
\text{bottom} &=&-&\dfrac{\text {canvas size}}{2}\\
\text{left} &=&-&\dfrac{\text {canvas size}}{2}\\
\end{array}
$$

Once we know the canvas bottom-left and top-right canvas coordinates, we can then compare the projected point coordinates with these values (we, of course, first need to compute the coordinates of the point onto the image plane, which is positioned at the near clipping plane. We will learn how to do so in the next chapter). Points lie within the canvas boundary (and are therefore visible) if their x and y coordinates are either greater or equal and lower or equal than the canvas bottom-left and top-right canvas coordinates, respectively. The following code fragment computes the canvas coordinates and tests the coordinates of a point lying on the image plane against these coordinates:

```
float canvasSize = 2 * tan(angleOfView * 0.5) * Znear;
float top = canvasSize / 2;
float bottom = -top;
float right = canvasSize / 2;
float left = -right;
// compute projected point coordinates
Vec3f Pproj = ...;
if (Pproj.x < left || Pproj.x > right || Pproj.y < bottom || Pproj.y > top) {
    // point outside canvas boundaries. It is not visible.
}
else {
    // point inside canvas boundaries. Point is visible
}
```

The above equations conclude that given the camera's *angle of view* and the *distance to canvas*, it is possible to check if a projected point lies within the canvas boundaries.

## Camera to World and World to Camera Matrix

![Figure 8: transforming the camera coordinate system with the camera-to-word transformation matrix.](/images/perspective-matrix/camera2.png?)

Finally, we need a method to produce images of objects or scenes from any viewpoint. We discussed this topic in the previous lesson, but we will cover it briefly in this chapter. CG cameras are similar to real cameras in that respect. However, in CG, we look at the camera's view (the equivalent of a real camera viewfinder) and move around the scene or object to select a viewpoint ("viewpoint" is the camera position in relation to the subject).

When a camera is created, by default, it is located at the origin and oriented along the negative z-axis (Figure 8). This orientation is explained in detail in the [previous lesson](/lessons/3d-basic-rendering/computing-pixel-coordinates-of-3d-point/mathematics-computing-2d-coordinates-of-3d-points). By doing so, the camera's local and world coordinate system's x-axis point in the same direction. Therefore, defining the camera's transformations with a 4x4 matrix is convenient. This 4x4 matrix which is no different from 4x4 matrices used to transform 3D objects, is called the **camera-to-world** transformation matrix (because it defines the camera's transformations with respect to the world coordinate system).

The camera-to-world transformation matrix is used differently depending on whether rasterization or ray tracing is being used:

- In rasterization, the inverse of the matrix (the world-to-camera 4x4 matrix) is used to convert points defined in world space to camera space. Once in camera space, we can perform a perspective divide to compute the projected point coordinates in the image plane. An in-depth description of this process can be found in the previous lesson.
- In ray tracing, we build camera rays in the camera's default position (the rays' origin and direction) and then transform them with the camera-to-world matrix. The full process is detailed in the "Ray-Tracing: Generating Camera Rays" lesson.

Don't worry if you still need to understand how ray tracing works. We will study rasterization first and then move on to ray tracing next.

## Understanding How Virtual Cameras Are Used

At this point of the lesson, we have explained almost everything there is to know about pinhole cameras and CG cameras. However, we still need to explain how images are formed with these cameras. The process depends on whether the rendering technique is rasterization or ray tracing. We are now going to consider each case individually.

![Figure 9: in the real world, when the light from a light source reaches an object, it is reflected into the scene in many directions. Only one ray goes in the camera's direction and strikes the film or sensor.](/images/cameras/pinholecam3.png?)

![Figure 10: each ray reflected off of the surface of an object and passing through the aperture, strikes a pixel.](/images/cameras/pinholecam2.png?)

Before we do so, let's briefly recall the principle of a pinhole camera again. When light rays emitted by a light source intersect objects from the scene, they are reflected off of the surface of these objects in random directions. For each point of the scene visible by the camera, only one of these reflected rays will pass through the aperture of the pinhole camera and strike the surface of the photographic paper (or film or sensor) in one unique location. If we divide the film's surface into a regular grid of pixels, what we get is a **digital pinhole camera**, which is essentially what we want our virtual camera to be (Figures 9 and 10).

This is how things work with a real pinhole camera. But how does it work in CG? In CG, cameras are built on the principle of a pinhole camera, but the image plane is in front of the center of projection (the aperture, which in our virtual camera model we prefer to call the eye), as shown in Figure 11. How the image is produced with this virtual pinhole camera model depends on the rendering technique. First, let's consider the two main visibility algorithms: rasterization and ray tracing.

### Rasterisation

![Figure 11: perspective projection of 3D points onto the image plane.](/images/cameras/pinholecam4.png?)

![Figure 12: perspective projection of a 3D point onto the image plane.](/images/cameras/pinholecam5.png?)

We will have to explain how the rasterization algorithm works in this chapter. To have a complete overview of the algorithm, you are invited to read the lesson devoted to the REYES algorithm, a popular rasterization algorithm. Next, we will examine how the pinhole camera model is used with this particular rendering technique. To do so, let's recall that each ray passing through the aperture of a pinhole camera strikes the film's surface in one location, which is eventually a pixel if we consider the case of digital images. 

Let's take the case of one particular ray, R, reflected off of the surface of an object at O, traveling towards the eye in the direction D, passing through the aperture of the camera in A, and striking the image at the pixel location X (Figure 12). To simulate this process, all we need to do is compute in which pixel of an image any given light ray strikes the image and record the color of this light ray (the color of the object at the point where the ray was emitted from, which in the real world, is essentially the information carried by the light ray itself) at that pixel location in the image.

This is the same as calculating the pixel coordinates X of the 3D point O using perspective projection. In perspective projection, the position of a 3D point onto the image plane is found by computing the intersection of a line connecting the point to the eye with the image plane. The method for computing this point of intersection was described in detail in the [previous lesson](/lessons/3d-basic-rendering/computing-pixel-coordinates-of-3d-point/mathematics-computing-2d-coordinates-of-3d-points). In the next chapter; we will learn how to compute these coordinates when the canvas is positioned at an arbitrary distance from the eye (in the previous lesson, the distance between the eye and the canvas was always assumed to be equal to 1).

Don't worry too much if you need help understanding clearly how rasterization works at this point. As mentioned before, a lesson is devoted to this topic alone. The only thing you need to remember from that lesson is how we can "project" 3D points onto the image plane and compute the projected point pixel coordinates. Remember that this is the method that we will be using with rasterization. The projection process can be seen as an interpretation of the way an image is formed inside a pinhole camera by "following" the path of light rays from whether points they are emitted from in the scene to the eye and "recording" the position (in terms of pixel coordinates) where these light rays intersect the image plane. To do so, we first need to transform points from world space to camera space, perform a perspective divide on the points in camera space to compute their coordinates in screen space, then convert the points' coordinates in screen space to NDC space, and finally convert these coordinates from NDC space to raster space. We used this method in the previous lesson to produce a wireframe image of a 3D object.

```
for (each point in the scene) {
    transform a point from world space to camera space;
    perform perspective divide (x/-z, y/-z);
    if (point lies within canvas boundaries) {
        convert coordinates to NDC space;
        convert coordinates from NDC to raster space;
        record point in the image;
    }
}
// connect projected points to recreate the object's edges
...
```

In this technique, the image is formed by a collection of "points" (these are not points, but conceptually, it is convenient to define where the light rays are reflected off the objects' surface as points) projected onto the image. In other words, you start from the geometry, and you "cast" light paths to the eye, to find the pixel coordinates where these rays hit the image plane, and from the coordinates of these intersections points on the canvas, you can then find where they should be recorded in the digital image. So, in a way, the rasterization approach is "object-centric".

### Ray-Tracing

![Figure 13: the direction of a light ray R can be defined by tracing a line from point O to the camera's aperture A or from the camera's aperture A to the pixel X, the pixel struck by the ray.](/images/cameras/pinholecam6.png?)

![Figure 14: the ray-tracing algorithm can be described in three steps. First, we build a ray by tracing a line from the eye to the center of the current pixel. Then, we cast this ray into the scene and check if this ray intersects any geometry in the scene. If it does, we set the current pixel's color to the object's color at the intersection point. This process is repeated for each pixel in the image.](/images/cameras/pinholecam7.png?)

The way things work in ray tracing (with respect to the camera model) is the opposite of how the rasterization algorithm works. When a light ray R reflected off of the surface of an object passes through the aperture of the pinhole camera and hits the surface of the image plane, it hits a particular pixel X on the image, as described earlier. In other words, each pixel, X, in an image corresponds to a light ray, R, with a given direction, D, and a given origin O. Note that we do not need to know the ray's origin to define its direction. The ray's direction can be found by tracing a line from O (the point where the ray is emitted) to the camera's aperture A. It can also be defined by tracing a line from pixel X where the ray intersects the camera's aperture A (as shown in Figure 13). Therefore, if you can find the ray direction D by tracing a line from X (the pixel) to A (the camera's aperture), then you can extend this ray into the scene to find O (the origin of the light ray) as shown in Figure 14. This is the ray tracing principle (also called ray casting). We can produce an image by setting the pixel's colors with the color of the light rays' respective points of origin. Due to the nature of the pinhole camera, each pixel in the image corresponds to one singular light ray that we can construct by tracing a line from the pixel to the camera's aperture. We then cast this ray into the scene and set the pixel's color to the color of the object the ray intersects (if any — the ray might not intersect any geometry indeed, in which case we set the pixel's color to black). This point of intersection corresponds to the point on the object's surface, from which the light ray was reflected off towards the eye.

Contrary to the rasterization algorithm, ray tracing is "image-centric". Rather than following the natural path of the light ray, from the object to the camera (as we do with rasterization in a way), **we follow the same path but in the other direction**, from the camera to the object.

In our virtual camera model, rays are all emitted from the camera origin; thus, the aperture is reduced to a singular point (the center of projection); the concept of aperture size in this model doesn't exist. Our CG camera model behaves as an **ideal pinhole camera** because we consider that a single ray only passes through the aperture (as opposed to a beam of light containing many rays as with real pinhole cameras). This is, of course, impossible with a real pinhole camera. When the hole becomes too small, light rays are diffracted. With such an ideal pinhole camera, we can create perfectly sharp images. Here is the complete algorithm in pseudo-code:

```
for (each pixel in the image) {
    // step 1
    build a camera ray: trace line from the current pixel location to the camera's aperture;
    // step 2
    cast ray into the scene;
    // step 3
    if (ray intersects an object) {
        set the current pixel's color with the object's color at the intersection point;
    }
    else {
        set the current pixel's color to black;
    }
}
```

![Figure 15: the point visible to the camera is the point with the closest distance to the eye.](/images/cameras/pinholecam8.png?)

As explained in the first lesson, ray-tracing things are a bit more complex because any camera ray can intersect several objects, as shown in Figure 15. Of all these points, the point visible to the camera is the closest distance to the eye. Suppose you are interested in a quick introduction to the ray-tracing algorithm. In that case, you can read the [first lesson](/lessons/3d-basic-rendering/introduction-to-ray-tracing/) of this section or keep reading the lessons from this section devoted to ray-tracing specifically.

Advanced: it may have come to your mind that several rays may be striking the image at the same pixel location. This idea is illustrated in the adjacent image. This happens all the time in the real world because the surfaces from which the rays are reflected are continuous. In reality, we have the projection of a continuous surface (the surface of an object) onto another continuous surface (the surface of a pixel). It is important to remember that a pixel in the physical world is not an ideal point but a surface receiving light reflected off from another surface. It would be more accurate to see the phenomenon (which we often do in CG) as **an "exchange" or transport of light energy between surfaces**. You can find information on this topic in lessons from the Mathematics and Physics of Compute Graphics (check the Mathematics of Shading and Monte Carlo Methods) as well as the lesson called Monte Carlo Ray Tracing and Path Tracing.

## What's Next?

We are finally ready to implement a pinhole camera model with the same controls as the controls you can find in software such as Maya. It will be followed as usual with the source code of a program capable of producing images matching the output of Maya.