## Introducing Coordinate Systems

Coordinate systems play an essential role in the graphics pipeline. They are not complicated; coordinates are one of the first things we learn in school when we study geometry. However, learning a few things about them will make understanding matrices easier.

In the previous chapter, we mentioned that points and vectors (as used in CG) are represented with three real numbers. But what do these numbers mean? Each number represents a **signed distance** from the origin of a line to the position of the point on that line. For example, consider drawing a line and putting a mark in the middle. We will call this mark the **origin**. This mark becomes our point of reference: the position from which we will measure the distance to any other point. If a point lies to the right side of the origin, we take the signed distance to be greater than zero (that is, positive something). On the other hand, if it is on the left side of the origin, the value will be negative (negative something).

We assume the line goes to infinity on either side of the origin. Therefore, the distance between two points on that line could be infinitely large. However, this presents a problem: in the world of computers, there is a practical limit to the value you can represent for a number (which depends on the number of bits we use to encode that number). Thankfully, this maximum value is usually big enough to build most of the 3D scenes we want to render; all the values we deal with in CG are bounded anyway. With that said, let's not worry too much about this computational limitation for now.

Now that we have a line and an origin, we add some additional marks at a regular interval (unit length) on each side of the origin, effectively turning our line into a ruler. With the ruler established, we can measure the **coordinate** of a point from the origin ("coordinate" is another way of saying the signed distance from the origin to the point). In computer graphics and mathematics, the ruler defines what we call an **axis**.

![Figure 1: the position of a point is defined as the (signed) distance from the point's position to the origin of the axis. The axis extends from minus to plus infinity.](/images/geometry/oneaxis.png?)

Suppose the point we are interested in is not on the axis. In that case, we can still find the point's coordinate by projecting it onto the ruler using a vertical line (Assuming the ruler is horizontal. Generally, we use a line perpendicular to the ruler). The distance from the origin to the intersection of this vertical line with the ruler is the coordinate of that point with respect to that axis. We have just learned to define the coordinate of a point along an axis.

## Dimensions and Cartesian Coordinate Systems

Let's call the horizontal ruler from before the **x-axis.** We can draw another ruler perpendicular to the x-axis at its origin. We will call this the **y-axis**. For any point, we can determine the x- and y-coordinate by drawing perpendicular lines to each axis and measuring the distance from those intersections to the origin (the same processes described above). We can now find two numbers, or two coordinates, for an arbitrary point: one for the x-axis and one for the y-axis. Thus, we have defined a two-dimensional space called a plane by placing two axes.

For example, consider drawing several points on a piece of paper. This piece of paper occupies a two-dimensional space, i.e., a plane. We can again draw two axes: one for each dimension. If we use the same x- and y-axes to measure each point drawn on that paper, these two axes are said to define a coordinate system. If these two rulers are perpendicular to each other, they represent a **Cartesian coordinate system**.

Note that we commonly use a concise notation called an **ordered pair** to write the coordinates of a point. An ordered pair is simply two numbers separated by a comma. For Cartesian coordinate systems, it is customary first to write the horizontal x-coordinate followed by the vertical y-coordinate. So, for example, we would write (2.5, 2.25) for a point whose x-coordinate was 2.5 and whose y-coordinate was 2.25 (see Figure 2). However, do not let this intimidate you. Remember, we can always interpret these ordered pairs as two signed distances: the point is 2.5 units right of and 2.25 units up from the origin. We will use this way of writing a point's coordinates in the coming lessons.

![Figure 2: a 2D Cartesian coordinate system is defined by two perpendicular right-angled axes (represented by the grey square in the middle of the figure). Each axis is divided into regular intervals of unit length. Computing the coordinates of a 2D point is simply an extension of the 1D case (in Figure 1) to the 2D case. We take signed distances from the point to the origin of the coordinate system in both x and y.](/images/geometry/axis2d.png?)

At this point in the lesson, we now know how to make a two-dimensional Cartesian coordinate system and define the coordinates of a 2D point in that coordinate system. Note that the coordinates of points defined in a coordinate system are unique. This means that the same point cannot be represented simultaneously by two different sets of coordinates in one system. However, we must note that we can choose any coordinate system.

We can choose to define infinitely many such coordinate systems in a plane. For the sake of simplicity, let's assume that we drew just two such Cartesian coordinate systems on a sheet of paper. In this paper, we place one point. The coordinates of that point will differ depending on which of the two coordinate systems we consider. For instance, in Figure 3, point P has coordinates (-1, 3) in coordinate system A and (2, 4) in coordinate system B. However, this is the same point; the dot is in the same place as when we first drew it.

![Figure 3: the same point is defined in two coordinate systems. We can transform the point in the red coordinate system (A) to the green coordinate system (B) by adding the values (3, 1) to its coordinates.](/images/geometry/pointincoordsystems.png?)

![Figure 4: scaling (shown in blue) or translating a point (shown in green) modifies its coordinates. A scale is a multiplication of the point's coordinates by some value. A translation is an addition of some values to the point's coordinates.](/images/geometry/scalepoint.png?)

So if you know the coordinates of P in coordinate system A, what do you need to do to find the coordinate of the same point in another coordinate system, B? This represents an essential operation in CG (and mathematics in general). We will soon learn why along with how to find the map which translates the coordinates of a point from one coordinate system to another (check the chapter [Transforming Points and Vectors](/lessons/mathematics-physics-for-computer-graphics/geometry/transforming-points-and-vectors)).

For now, let's consider the previous example in Figure 3. Note that adding the values (3, 1) coordinate-wise (that is, add the two x-axis values and then add the two y-axis values independently) to the coordinates (-1, 3) leads to the coordinate (2, 4). So adding (3, 1) to the coordinates of P in A yields the coordinates of P in B. Adding (-3, -1) to (2, 4) yields (-1, 3). This takes the coordinates of P in B to the coordinates of P in A. It is important to note that (-3, -1) is just the additive inverse (or the opposite) of (3, 1). This is intuitive as they can be thought of as altering a point's coordinates in opposite directions: adding (3, 1) maps in one direction, from A to B, whereas (-3, -1) maps in the opposite, inverse direction from B to A.

Another common operation is to move the point in the coordinate system A to another location in the same coordinate system. This is called a **translation** and is undoubtedly one of the most basic operations you can do on points. Note that all linear operators can be applied to point coordinates. For example, a multiplication of a real number to the coordinates of a point produces a **scale** (figure 4). A scale moves P along the line that goes through the point and the origin (because when we transform a point, we transform the vector from the origin to the point). More on all of this later.

## The Third Dimension

The 3D coordinate system is a simple extension of the 2D case. We will add a third axis orthogonal to both the x- and y-axis called the z-axis (representing depth). The x-axis points to the right, the y-axis points up, and the z-axis points backward (it comes out of the screen in a way when the x-axis points to the right). While other conventions can be used (see the following paragraph), we will only use this throughout Scratchapixel. In Geometry, this 3D coordinate system defines what is more formally known as **Euclidean space**.

![Figure 5: a three-dimensional coordinate system. Three coordinates define a point, one for each axis.](/images/geometry/coordsys3d.png)

We conclude this chapter with a paragraph for those interested in a more formal coordinate system definition. In linear algebra, the three axes (one or two in the 1D and 2D cases, respectively) form what we call the **basis** of that coordinate system. A basis is a set of linearly independent vectors that, in a linear combination, can represent every vector (or point) in a given vector space (the coordinate system). Vectors from a set are said to be linearly independent if and only if none of the vectors in the set can be written as a linear combination of other vectors in that set. **Change of basis**, or change of coordinate system, is a common operation in mathematics and the graphics pipeline.

## Left-Handed vs. Right-Handed Coordinate Systems

Unfortunately, due to various conventions concerning handedness, coordinate systems are more complex. The problem can be illustrated in the following figure. When the y-axis (also called the "up" vector) points upward, and the x-axis (also called the "right" vector) points to the right (as depicted in the image below), the z-axis (also called the forward or down vector) can point either toward the screen or you. Think of your screen as the XY plane. Then the forward vector (z-axis) would be perpendicular to your screen. Look at the z-axis as indicating the direction of depth.

![](/images//geometry/geo-lh-vs-rh-coordsys.png)

To differentiate the two conventions, we call the first coordinate system the **left-hand coordinate system**, and the other the **right-hand coordinate system**. The physicist John Ambrose Fleming introduced the left- and right-hand rule to differentiate the two conventions easily.

We will explain what it has to do with hands in a moment. But first, let's define the difference between the two systems. With the x-axis pointing to the right and the y-axis pointing up, if the z-axis points away from you, it's a left-hand coordinate system. If it points in your direction, it is a right-hand coordinate system.

!!!
To remember each coordinate system's orientation, assign the x, y, and z-axis to your thumb, index, and middle finger, respectively. This is super simple because this is the order by which we naturally unfold our fingers. Do this with your left and right hand. Then orient the thumb to the right, the index upward, and your middle finger will indicate the direction of the z-axis.

- Left hand: away from you.

- Right hand: toward you.

This mnemonic requires a bit of hand contortion, but it is the best way of getting it "right" all the time. So now you understand why we call these systems the left-hand and right-hand coordinate systems or speak of the coordinate system's handedness.
!!!

![Figure 6: orient your left or right hand as depicted in this figure to find out in which direction the z-axis points to when you the left or right hand coordinate systems are used respectively. Hand 3D model courtesy of Artem Manuilov.](/images//geometry/geo-lefthand-vs-righthand.png)

<details>
With the thumb pointing upward (y-axis), and the index finger (z-axis) pointing away from you, in the left-hand coordinate system, the middle finger (x-axis) points to the right; in the right-hand coordinate system, it points to the left. 

![](/images/geometry/rhlh.png)

Here the left and right hands are just used to figuring out in which direction the x-axis points, assuming the thumb indicates the up vector and the index finger the forward vector, which indeed points forward (with respect to the viewer). With this mnemonic, the x-axis (or third vector) points to the right when the left hand is used (and to the left when the right hand is used). We understand that the left and right terms here refer to the use of your left or right hand, not the direction of the x-axis. Having to say that your middle finger points to the right in the left-hand coordinate system is counter-intuitive. That's why we prefer the XYZ-thumb-index-middle finger mnemonic mentioned above.

We are mentioning it nonetheless because you might find it used in other documents (in fact, this is how Scratchapixel initially explained the difference between the two systems).
</details>

The handedness of the coordinate system also plays a role in the orientation of normals computed from the edges of polygonal faces. For example, if the direction is right-handed, polygons whose vertices were specified in the **counterclockwise** order will be front-facing. This will be explained in the lesson on rendering polygonal objects.

## The Right, Up, and Forward Vectors

![Figure 7: the most popular convention used in CG defines the up vector as being the y-axis (a). However, it's common to find coordinate systems in many CG-related papers (particularly those related to shading techniques) where the up vector is defined as the z-axis (b). Some authors claim this convention comes from the notation commonly used in physics and mathematics. Both coordinate systems drawn in this figure are right-handed.](/images/geometry/zaxis.png?)

Three perpendicular vectors of unit length only define the Cartesian coordinate system. Regarding the mathematical notation, **this coordinate system needs to convey more about what these three axes mean**. The developer is the one that decides how these axes should be interpreted. It is thus essential to make a clear distinction between the handedness of the coordinate system and the conventions used to label the corresponding axes.

Is the up vector called the z-axis or y-axis? Let's use the convention from Figure 7b and assume that the x-axis is the right vector. What can we say about the handedness of this coordinate system? It's a right-handed coordinate system (orient the middle finger of your right hand along the x-axis and check if the other two fingers points along the up and forward vector). As you can see, we take either a right- or left-handed coordinate system and label the axes x, y and z. The naming convention (how you label these axes) has nothing to do with the handedness of the coordinate system. It is crucial to understand this difference. Many people often think that because some system uses a convention where the up vector is **labeled** the z-axis (instead of the more popular y-axis convention), one system is left-handed, and the other is right-handed. Not at all.

**The only thing that defines the handedness of the coordinate system is the orientation of the left (or right) vector relative to the up and forward vectors, regardless of what these axes represent.** Handedness and conventions regarding the names of the axes are two different things.

Knowing which convention is used for the coordinate system when dealing with a renderer or any other 3D application is also essential. Currently, the industry standard tends to be the right-hand XYZ coordinate system where x points to the right, y is up, and z is outward (coming out of the screen). Programs and 3D APIs such as Maya and OpenGL use a right-hand coordinate system, while DirectX, PBRT, and PRMan use a left-hand coordinate system. Note that Maya and PRMan use a coordinate system in which the up vector is called the y-axis, and the forward vector is called the z-axis. This means that the z-coordinate of 3 for a point in one system is -3 in the other. For this reason, we may need to reverse the sign of an object's z-coordinates when the geometry is exported to the renderer. The choice of coordinate system handedness also plays a critical role in the two vectors' rotation and cross-product. We will talk about this more in the following few chapters. It's easy enough (but painful) to go from one coordinate system to another. All that is needed is to scale the point coordinates and the camera-to-world matrix by (1, 1, -1).

For now, note that **Scratchapixel uses a right-hand coordinate system** mainly to stay compatible with Maya and because it has become the de-facto industry standard anyway (we wished everybody was using the same conventions).

## The World Coordinate System

We have learned that points' and vectors' coordinates relate to the origin of a Cartesian coordinate system defined by three perpendicular unit vectors (that make up a basis). We have also explained that we can create as many coordinate systems as we want and that points and vectors have unique coordinates within each of these coordinate systems. However, in most 3d applications, each type of coordinate system is defined with respect to a master coordinate system called the **world** coordinate system. It represents the origin and the main x-, y-, and z-axes defined by all other coordinate systems. The world coordinate system is the most important of all the different coordinate systems in the rendering pipeline. These include the object, local (used in shading), camera, and screen coordinate systems. We will explain all of these as we go along.

## Things We Need To Remember

We realize that most readers (if not all) do not need to have these concepts explained. However, what is important here is not so much knowledge of basic geometry but being comfortable with the terminology used throughout all CG literature. In this chapter, the terms of importance are **coordinates**, **axes**, and **Cartesian coordinate system**. We have also introduced the concept of linear operators (scale and translate) to transform points and vectors. The essential ideas to remember from that chapter are that points' coordinates relate to a coordinate system, a multitude of coordinate systems can be defined and that a point has unique coordinates in each of these coordinate systems. Determining whether the coordinate system you use (either in your program or in the API you use to render images) is left- or right-handed is also essential. Keeping the handedness of a coordinate system distinct from the convention used for labeling the axes is essential.