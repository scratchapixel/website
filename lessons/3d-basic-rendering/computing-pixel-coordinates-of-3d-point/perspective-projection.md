## How Do I Find the 2D Pixel Coordinates of a 3D Point?

"**How do I find the 2D pixel coordinates of a 3D point?**" is one of the most common questions in 3D rendering on the Web. It is an essential question because it is the fundamental method to create an image of a 3D scene. In this lesson, we will use the term **rasterization** to describe the process of finding 2D pixel coordinates of 3D points. In its broader sense, Rasterization refers to converting 3D shapes into a raster image. A raster image, as explained in the [previous lesson](/lessons/3d-basic-rendering/rendering-3d-scene-overview/), is the technical term given to a digital image; it designates a two-dimensional array (or rectangular grid if you prefer) of pixels.

Don't be mistaken: different rendering techniques exist for producing images of 3D scenes. Rasterization is only one of them. Ray tracing is another. Note that all these techniques rely on the same concept to make that image: the idea of **perspective projection**. Therefore, for a given camera and a given 3D scene, all rendering techniques produce the same visual result; they use a different approach to produce that result.

Also, computing the 2D pixel coordinates of 3D points is only one of the two steps in creating a photo-realistic image. The other step is the process of shading, in which the color of these points will be computed to simulate the appearance of objects. You need more than just converting 3D points to pixel coordinates to produce a "complete" image.

![](/images/perspective-matrix/xtree.png?)

To understand rasterization, you first need to be familiar with a series of essential techniques that we will also introduce in this chapter, such as:
- The concept of local vs. global coordinate system.
- Learning how to interpret 4x4 matrices as coordinate systems.
- Converting points from one coordinate system to another.

Read this lesson carefully, as it will provide you with the fundamental tools that almost all rendering techniques are built upon.

We will use matrices in this lesson, so read the Geometry lesson if you are uncomfortable with coordinate systems and matrices.

We will apply the techniques studied in this lesson to render a **wireframe** image of a 3D object (adjacent image). The files of this program can be found in the source code chapter of the lesson, as usual.

## A Quick Refresher on the Perspective Projection Process

![Figure 1: to create an image of a cube, we need to extend lines from the corners of the object towards the eye and find the intersection of these lines with a flat surface (the canvas) perpendicular to the line of sight.](/images/rendering-3d-scene-overview/perspective4.png?)

We talked about the perspective projection process in quite a few lessons already. For instance, check out the chapter [The Visibility Problem](/lessons/3d-basic-rendering/rendering-3d-scene-overview/visibility-problem) in the lesson "Rendering an Image of a 3D Scene: an Overview". However, let's quickly recall what perspective projection is. In short, this technique can be used to create a 2D image of a 3D scene by projecting points (or vertices) that make up the objects of that scene onto the surface of a canvas.

We use this technique because it is similar to how the human eye works. Since we are used to seeing the world through our eyes, it's pretty natural to think that images created with this technique will also look natural and "real" to us. You can think of the human eye as just a "point" in space (Figure 2) (of course, the eye is not exactly a point; it is an optical system converging rays onto a small surface - the retina). What we see of the world results from light rays (reflected by objects) traveling to the eye and entering the eye. So again, one way of making an image of a 3D scene in computer graphics (CG) is to do the same thing: project vertices onto the surface of the canvas (or screen) as if the rays were sliding along straight lines that connect the vertices to the eye.

It is essential to understand that perspective projection is just an arbitrary way of representing 3D geometry onto a two-dimensional surface. This method is most commonly used because it simulates one of the essential properties of human vision called **foreshortening**: objects far away from us appear smaller than objects close by. Nonetheless, as mentioned in the Wikipedia article on [perspective](https://en.wikipedia.org/wiki/Perspective_(graphical)), it is essential to understand that the perspective projection is only an **approximate representation** of what the eye sees, represented on a flat surface (such as paper). The important word here is "approximate".

![Figure 2: among all light rays reflected by an object, some of these rays enter the eye, and the image we have of this object, is the result of these rays.](/images/perspective-matrix/raystoeye.png?)

![Figure 3: we can think of the projection process as moving a point down along the line that connects the point to the eye. We can stop moving the point along that line when the point lies on the plane of the canvas. We don't explicitly "slide" the point along this line, but this is how the projection process can be interpreted.](/images/rendering-3d-scene-overview/projection3.png?)

In the lesson mentioned above, we also explained how the world coordinates of a point located in front of the camera (and enclosed within the viewing frustum of the camera, thus visible to the camera) could be computed using a simple geometric construction based on one of the properties of similar triangles (Figure 3). We will review this technique one more time in this lesson. The equations to compute the coordinates of projected points can be conveniently expressed as a 4x4 matrix. The computation is simple but a series of operations on the original point's coordinates: this is what you will learn in this lesson. However, by expressing the computation as a matrix, you can reduce these operations to a single point-matrix multiplication. This approach's main advantage is representing this critical operation in such a compact and easy-to-use form. It turns out that the perspective projection process, and its associated equations, can be expressed in the form of a 4x4 matrix, as we will demonstrate in the lesson devoted to the [the perspective and orthographic projection matrices](/lessons/3d-basic-rendering/perspective-and-orthographic-projection-matrix/). This is what we call the **perspective projection matrix**. Multiplying any point whose coordinates are expressed with respect to the **camera coordinate system** (see below) with this perspective projection matrix will give you the position (or coordinates) of that point on the canvas.

<details>
In CG, transformations are almost always linear. But it is essential to know that the perspective projection, which belongs to the more generic family of **projective transformation**, is a non-linear transformation. If you're looking for a visual explanation of which transformations are linear and which transformations are not, this [Youtube video](https://www.youtube.com/watch?v=kYB8IZa5AuE) does a good job.
</details>

Again, in this lesson, we will learn about computing the 2D pixel coordinates of a 3D point without using the perspective projection matrix. To do so, we will need to learn how to "project" a 3D point onto a 2D drawable surface (which we will call in this lesson a canvas) using some simple geometry rules. Once we understand the mathematics of this process (and all the other steps involved in computing these 2D coordinates), we will then be ready to study the construction and use of the perspective projection matrix: a matrix used to simplify the projection step (and the projection step only). This will be the topic of the next lesson.

## Some History

![](/images/perspective-matrix/duerer.png?)

The mathematics behind perspective projection started to be understood and mastered by artists towards the end of the fourteenth century and the beginning of the fifteenth century. Artists significantly contributed to educating others about the mathematical basis of perspective drawing through books they wrote and illustrated themselves. A notable example is "The Painter's Manual" published by Albrecht Dürer in 1538 (the illustration above comes from this book). Two concepts broadly characterize perspective drawing:

- Objects appear smaller as their distances to the viewer increase.
- **Foreshortening**: the impression, or optical illusion, that an object or a distance is smaller than it is due to being angled towards the viewer.

Another rule in foreshortening states that vertical lines are parallel, while nonvertical lines converge to a perspective point, appearing shorter than they are. These effects give a sense of depth, which helps evaluate the distance of objects from the viewer. Today, the same mathematical principles are used in computer graphics to create a perspective view of a 3D scene.
