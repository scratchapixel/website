## What You Need to Know First

Before we start studying how to build a basic perspective projection matrix, we first need to review some of the techniques projection matrices are built upon.

## Converting the Perspective Frustum to a Unit Cube

![Figure 1: P' is the projection of P on the canvas.](/images/perspective-matrix/perspprojmatrix2.png?)

Multiplying a point P by our simple perspective projection matrix will give a point P' whose:

- x'- and y'-coordinates are the coordinates of P on the image plane. Both x' and y' are defined in NDC space. As mentioned in the introduction, the perspective projection matrix remaps a 3D point's coordinates to its "2D" position on the screen in NDC space (in the range [-1,1] in this lesson). Normally the matrix guarantees that points visible through the camera (contained in the frustum) are remapped to the range [-1,1] (regardless of whether or not the canvas is a square - these are not screen space coordinates but NDC coordinates).
- In addition to remapping the 3D point to its 2D coordinates, we will also need to remap its z-coordinate. In the previous lesson on rasterization, we didn't bother remapping z' at all, but GPUs do remap P' z-coordinate to the range [0,1] or [-1,1] depending on the API. When P lies on the near-clipping plane, z' is remapped to 0 (or -1), and when P lies on the far-clipping plane, z' is remapped to 1.

![Figure 2: the projection matrix remaps the viewing frustum into a unit cube or canonical view volume.](/images/perspective-matrix/canonical1.png?)

The fact that the x- and y-coordinates of P' as well as its z-coordinate are remapped to the range [-1,1] and [0,1] (or [01,1]) essentially means that the transformation of a point P by a projection matrix remaps the volume of the viewing frustum to a cube of dimension 2x2x1 (or 2x2x2). This cube is often referred to as the **unit cube** (it is not exactly a cube when it has dimension 2x2x1 but you get the idea) or **canonical view volume**. You can also see this process as if the viewing frustum was being normalized. This is a very important concept in CG which is sometimes hard to even visualize but the viewing frustum which is defined by the near and far clipping planes as well as the screen dimensions which is not a cube at all but has the shape of a truncated pyramid, is indeed "warped" into a cube. This is essentially what the projection matrix does. When the space defined by the frustum is "warped" into a cube, it then becomes easier to operate on points (a cube is a much easier geometrical form to work with than a truncated pyramid). This concept is very important and is one of the things you should remember about projection matrices. A projection matrix (at least the way it is used in CG) remaps the space defined by the viewing frustum to a unit cube.

### About Clipping

![Figure 3: example of clipping in 2D. At the clipping stage, new triangles may be generated wherever the original geometry overlaps the boundaries of the viewing frustum.](/images/perspective-matrix/clipping2.png?)

![Figure 4: a point located behind the camera will be projected as are points in front, but its projected coordinates will mirrored in both directions](/images/perspective-matrix/clipping4.png?)

Let's introduce the concept of clipping. What it does is "clip" geometry straddling the boundaries of the viewing frustum. In other words, if some triangles or lines overlap the viewing frustum planes, the geometry is "chopped off" in such a way that the parts of the geometry that are contained within the frustum are kept and the parts which are outside the frustum volume (and thus not visible to camera), discarded (figure 3). Clipping may only seem like an optimization process, though its main goal is not to throw away parts of the scene that are not visible to speed up the render. "Unfortunately" the perspective projection works equally well for objects which are in front or behind the camera. Consider a point located behind the "observer". Let's imagine a point whose coordinates are (2, 5, 10). If you apply the rules of perspective projection to this point you would get:

$$
\begin{array}{l}
x' = \dfrac{2}{-10} = -0.2,\\
y' = \dfrac{5}{-10} = -0.5,\\
\end{array}
$$

Note how the projected coordinates would be perfectly valid but note also that the point is mirrored on the canvas in both directions. While the x- and y-coordinates of the point in camera space are both positive, they end up being negative in screen space (figure 4).

A lesson to explain one of the most common clipping algorithms known as the [Cohen-Sutherland algorithm](http://en.wikipedia.org/wiki/Cohen?Sutherland_algorithm) will be later added to the section on advanced rasterization, though we will talk more about clipping and clip space in chapter four.

Now that we understand the concept of clipping though, we can more easily explain why this transformation from the viewing frustum to this canonical view volume is done.

![Figure 5: converting perspective frustum to unit cube before clipping.](/images/perspective-matrix/clipping3.png?)

- The main reason, as we mentioned before, is that it transforms a rather uneasy space to work with (the truncated pyramid of the viewing frustum) into a basic box. Within this space, operations such as clipping, for instance, can be done more easily.
- Once defined with respect to this canonical viewing volume, it becomes trivial to convert the points' 3D coordinates to 2D coordinates on the image plane.

Keep in mind that the model of camera we want to simulate is the **pinhole camera**, which is defined by a near and far clipping plane as well as an angle of view (see the lesson on the [pinhole camera model](/lessons/3d-basic-rendering/3d-viewing-pinhole-camera/)). The angle of view parameter will need to be taken into account when points are remapped from screen space to NDC space.

## Projecting Points onto the Screen

Before we study how to create a perspective matrix, we will first review one more time how to project 3D points onto the screen (this process is described in detail in the lesson [Computing the Pixel Coordinates of a 3D Point](/lessons/3d-basic-rendering/computing-pixel-coordinates-of-3d-point/)). Usually, 3D points being projected onto the image plane are first transformed into the camera coordinate system. In this coordinate system, the eye position corresponds to the origin, the x- and y-axes define a plane parallel to the image plane, and the z-axis is perpendicular to that XY plane. In our setup, the image plane will be located exactly one unit away from the origin of the camera coordinate system, i.e the eye. You might be confused by this convention if you are used to a system in which the distance to the image plane is arbitrary, which is the case in OpenGL. We will learn how to extend the matrix to handle arbitrary clipping planes in the next chapters. But for the time being, we will use this convention to simplify the demonstration.

![](/images/perspective-matrix/camsetup.png?)

<details>
Keep in mind that Scratchapixel uses a right-hand coordinate system, as do many other commercial applications, such as Maya. To learn more about right- and left-hand coordinate systems, check out the lesson on [Geometry](/lessons/mathematics-physics-for-computer-graphics/geometry/) in the Mathematics and Physics for Computer Graphics section. Because we use a right-hand coordinate system, the camera will be pointing in a direction opposite to the z-axis. This is because when we project points onto the image plane, we want the x-axis to point to the right. Mathematically speaking, all points visible to the camera have a negative z-component, when the points are expressed in the camera coordinate system. This is explained in detail in the [previous lesson](/lessons/3d-basic-rendering/computing-pixel-coordinates-of-3d-point/mathematics-computing-2d-coordinates-of-3d-points).
</details>

Let's imagine that we want to project a point P onto the canvas. If we draw a line from P to the eye, we can see that P is projected onto the screen at P'. How do we compute P'?

![Figure 6: To project P on the image plane (at P') we the XY coordinates of P by the z coordinate of P.](/images/perspective-matrix/projection4.png?)

In Figure 6, you can see that the green (\(\Delta ABC\)) and red (\(\Delta DEF\)) triangles have the same shape but not the same size. Such triangles are said to be **similar**. In other words, the red triangle can be seen as a scaled-down version of the green triangle. Similar triangles have a useful property: the ratio between their respective sides is constant. In other words:

$$\dfrac{BC} {EF} = \dfrac{AB} {DE}.$$

Since we are interested in the side of BC, i.e. the position of P' on the image plane, we can write:

$$BC = \dfrac{AB * EF} {DE}.$$

Given that B lies on the image plane, which is one unit away from A (AB = 1), we have our final formula for calculating the length of BC:

$$BC = \dfrac{(AB=1) * EF} {DE} = \dfrac{EF} {DE}.$$

From this equation, we can find the x- and y- coordinates of P'. All we need to do is divide the x- and y-coordinates of P by its z-coordinate. In mathematical form, we can write (equation 1):

$$
\begin{array}{l}
P'_x=\dfrac{P_x}{-P_z},\\
P'_y=\dfrac{P_y}{-P_z}.
\end{array}
$$

Note that we divided \(P_x\) and \(P_y\) by \(-P_z\) and not \(P_z\) because the z-component of the points visible through the camera is always negative when defined in the camera coordinate system. Computing the coordinates of P', which is the projection of P on the image plane, is thus really simple. Note that figure 6 only shows the projection of the y-coordinate of P on the image plane. If you rotate Figure 5 by ninety degrees clockwise and replace the y-axis with the x-axis, you will get a top view representing the projection of the x-coordinate of P onto the image plane.

## Homogeneous Coordinates

![Figure 7: to multiply a 3D point by a 4x4 matrix, we need to convert the point's Cartesian coordinates to homogeneous coordinates. Since all this conversion requires is to set the homogeneous fourth coordinate to 1, this conversion only needs to be implicit. Within the point-matrix multiplication function itself, we can convert the point from homogeneous back to Cartesian coordinates, by dividing the point transformed coordinates x', y' and z' by w'.](/images/perspective-matrix/homogeneous.png?)

You may think that there is nothing particularly complicated about perspective projection. The principle itself is rather simple indeed. The story, however, does not stop here. What we want is to encode this projection process into a matrix, so that projecting a point onto the image plane can be obtained via a basic point-matrix multiplication. Let's quickly review what we know about this process.

If you remember what we said in the lesson on [Geometry](/lessons/mathematics-physics-for-computer-graphics/geometry/transforming-points-and-vectors), two matrices can be multiplied with each other if the numbers on each side of the multiplication sign are equal or to say it differently, if the number of columns of the left-hand matrix and the number of rows of the right-hand matrix are equal.

$$
\begin{array}{l} 
{\color{\red} {\text{no:}}} &amp; [n \: m] * [q \: n] \\ 
{\color{\green} {\text{yes:}}} &amp; [m \: n]* [n \: q] \\ 
\end{array}
$$

Remember that a point can be represented by a one-row matrix (some people prefer the one-column notation, but Scratchapixel uses the one-row notation). But then our point is a 1x3 matrix (1 row, 3 columns) and therefore cannot be multiplied by a 4x4 matrix (4x4 matrices are used in CG to transform points and vectors. They encode rotation, scale, and translation). What can we do? To solve this problem, we employ a trick that consists of representing a point with not three by four coordinates. Such points are said to have **homogeneous coordinates** and can be represented in the form of a 1x4 matrix. The fourth coordinate of a point in its homogeneous representation is denoted by the letter **w**. When we convert a point from Cartesian to homogeneous coordinates, w is set equal to 1\. \(P_c\) (a point in Cartesian coordinates) and \(P_h\) (a point in homogeneous coordinates) are interchangeable, as long as w equals 1\. When w is different than 1, we **must** divide all four coordinates of the point [x y z w] by w to set the value of w back to 1 (if we wish to use the point as a 3D Cartesian point back again).

$$
\begin{array}{l}
[x \: y \: z] \neq [x \: y \: z \: w=1.2] \\
x=\dfrac{x}{w}, y=\dfrac{y}{w}, z=\dfrac{z}{w}, w=\dfrac{w}{w}=1 \\ 
{[x \: y \: z] = [x \: y \: z \: w = 1]}
\end{array}
$$

!!!
A maybe more formal way of defining this idea is to say that **the point with homogeneous coordinates [x, y, z, w] corresponds to the three-dimensional Cartesian point [x/w, y/w, z/w]**.
!!!

Here is what a typical transformation matrix looks like:

$$
\begin{bmatrix}
\color{green}{m_{00}} & \color{green}{m_{01}} & \color{green}{m_{02}} & \color{blue}{0}\\
\color{green}{m_{10}} & \color{green}{m_{11}} & \color{green}{m_{12}} & \color{blue}{0}\\
\color{green}{m_{20}} & \color{green}{m_{21}} & \color{green}{m_{22}} & \color{blue}{0}\\
\color{red}{T_x} & \color{red}{T_y} & \color{red}{T_z} & \color{blue}{1}\\
\end{bmatrix}
$$

The inner [3x3] matrix (in green) encodes rotation and scale. The three coefficients at the bottom of the matrix (in red) encode translation.

<details>
Remember that 4x4 transformation matrices are said to be affine. An affine transform has two very specific properties:

- Collinearity is preserved: all points lying on a line still lie on a line after the transformation is applied.
- Ratios of distances are preserved: the midpoint of a line segment remains the midpoint after the transformation is applied.

This is important to know because, in contrast, projective transforms (which we are going to introduce next) have the first property but not the second one. We already mentioned in the previous lesson that perspective projection preserves lines but not distances.
</details>

We know that we use these matrices to transform 3D points, however as we just said, what we do under the hood is treat these 3D points as if they were points with homogeneous coordinates. We do so by "implicitly" assuming that these 3D points have a fourth coordinate whose value is 1. Remember that a 3D point with coordinates {x, y, z} and a point with homogeneous coordinates {x, y, z, w} are equivalent as long as w = 1. A 3D point can be defined as a point with homogeneous coordinates if we write:

$$P = \{x, y, z, w = 1\}.$$

Always keep in mind that if you multiply a "3D" point by a 4x4 matrix, your point is (at least implicitly if you don't explicitly define this point with four coordinates which some programs do) a point with homogeneous coordinates and whose w-coordinate is equal to 1. Why we don't "explicitly" define points with four coordinates in programming, is simply to save memory (there's no point really to using memory for storing a coordinate whose value is always 1). Let's now multiply this 1x4 point by our 4x4 transformation matrix. If we multiply a [1x4] matrix (our point) by a [4x4] matrix, we should get a [1x4] matrix, in other words, another point with homogeneous coordinates. To convert that point back to 3D, we will need to divide the points' coordinates {x, y, z} by w. Though the fourth row of a 4x4 transformation matrix is **always** set to \(\color{blue}{{0, 0, 0, 1}}\) which means that as a result of the way points and matrices are multiplied with each other, \(\color{purple}{w'}\), the fourth coordinate of the transformed point is always equal to 1. Let's see why this happens:

$$
\begin{bmatrix}
x' & z' & y' & w'
\end{bmatrix}
=
\begin{bmatrix}
x & z & y & w = 1
\end{bmatrix}
*
\begin{bmatrix}
\color{green}{m_{00}} & \color{green}{m_{01}} & \color{green}{m_{02}} & \color{blue}{0}\\
\color{green}{m_{10}} & \color{green}{m_{11}} & \color{green}{m_{12}} & \color{blue}{0}\\
\color{green}{m_{20}} & \color{green}{m_{21}} & \color{green}{m_{22}} & \color{blue}{0}\\
\color{red}{T_x} & \color{red}{T_y} & \color{red}{T_z} & \color{blue}{1}\\
\end{bmatrix}
$$

$$
\begin{array}{l}
x' = x * m_{00} + y * m_{10} + z * m_{20} + (w = 1) * T_x,\\
y' = x * m_{01} + y * m_{11} + z * m_{21} + (w = 1) * T_y,\\
z' = x * m_{02} + y * m_{12} + z * m_{22} + (w = 1) * T_z,\\
\color{purple}{w' = x * 0 + y * 0 + z * 0 + (w = 1) * 1 = 1}.\\
\end{array}
$$

As you can see, regardless of the inner 3x3 matrix and translation coefficients value, \(\color{purple}{w'}\) will always be equal to 1. It is because \(\color{purple}{w'}\) is computed from w which is equal to 1, and the coefficients of the matrix fourth column, which for a transformation matrix, are always equal to \(\color{blue}{{0, 0, 0, 1}}\) respectively. They are constant. They never change otherwise this wouldn't be a transformation matrix (more likely a projection matrix as we will soon see). Practically, this also means that converting the x', y', and z'-coordinate back to Cartesian coordinates by dividing them by \(\color{purple}{w'}\) is not necessary since \(\color{purple}{w'}\) is also always equal to 1.

!!!
A 3D cartesian point P converted to a point with homogeneous coordinates {x, y, z, w = 1}, and multiplied by a 4x4 affine transformation matrix, always gives a point P' with homogeneous coordinates and whose w-coordinate \(\color{purple}{w'}\) is always equal to 1. Thus the conversion of the transformed point P' with homogeneous coordinates {x', y', z', w'} back to 3D Cartesian coordinate {x'/w', y'/w', z'/w'}, doesn't require an explicit normalization of the transformed point' homogeneous coordinates by \(\color{purple}{w'}\).
!!!

<details>
Technically, a function implementing a point multiplied by a 4x4 matrix should look like this (version 1):

```
template<typename S> 
void multVecMatrix(const Vec4<S> &src, Vec3<S> &dst) const 
{ 
    S a, b, c, w; 
 
    // note that src.w = 1
    a = src.x * x[0][0] + src.y * x[1][0] + src.z * x[2][0] + src.w * x[3][0]; 
    b = src.x * x[0][1] + src.y * x[1][1] + src.z * x[2][1] + src.w * x[3][1]; 
    c = src.x * x[0][2] + src.y * x[1][2] + src.z * x[2][2] + src.w * x[3][2]; 
    w = src.x * x[0][3] + src.y * x[1][3] + src.z * x[2][3] + src.w * x[3][3]; 
 
    dst.x = a / w; 
    dst.y = b / w; 
    dst.z = c / w; 
} 
```

It assumes that the source point is a point with homogeneous coordinates (hence the name Vec4) since only points with four coordinates can be multiplied by 4x4 matrices. In Vec4, the w-coordinate is explicitly defined. But since `src.w` is always assumed to be equal to 1 (this is the condition for points with Cartesian coordinates and points with homogeneous coordinates to be interchangeable), the code can be simplified to (version 2):

```
template<typename S> 
void multVecMatrix(const Vec3<S> &src, Vec3<S> &dst) const 
{ 
    S a, b, c, w; 
 
    // since src.w is assumed to always be equal to 1, it needn't be defined explicitly
    a = src.x * x[0][0] + src.y * x[1][0] + src.z * x[2][0] + x[3][0]; 
    b = src.x * x[0][1] + src.y * x[1][1] + src.z * x[2][1] + x[3][1]; 
    c = src.x * x[0][2] + src.y * x[1][2] + src.z * x[2][2] + x[3][2]; 
    w = src.x * x[0][3] + src.y * x[1][3] + src.z * x[2][3] + x[3][3]; 
 
    dst.x = a / w; 
    dst.y = b / w; 
    dst.z = c / w; 
} 
```

In this version, src is still a point with homogeneous coordinates, but since its w-coordinate is equal to 1, we don't need to explicitly define it, which is why src in this case is defined as a Vec3\. Furthermore, if the matrix is an affine transformation matrix, we know that w should also always be equal to 1\. Thus, w doesn't need to be computed and the division of the x, y, and z-coordinates by w can also be skipped. This reduces the code to (version 3):

```
template<typename S> 
void multVecMatrix(const Vec3<S> &src, Vec3<S> &dst) const 
{ 
    S a, b, c; 
 
    // since src.w is assumed to always be equal to 1, it needn't be defined explicitly
    a = src.x * x[0][0] + src.y * x[1][0] + src.z * x[2][0] + x[3][0]; 
    b = src.x * x[0][1] + src.y * x[1][1] + src.z * x[2][1] + x[3][1]; 
    c = src.x * x[0][2] + src.y * x[1][2] + src.z * x[2][2] + x[3][2]; 
 
    // no need to compute w explicitly. It's always equal to 1 for affine transforms
    //w = src.x * 0 + src.y * 0 + src.z * 0 + 1 * 1 = 1; 
 
    // division by w is not necessary
    dst.x = a; 
    dst.y = b; 
    dst.z = c; 
}
```

Which is the code that is typically used to transform points by affine transformation matrices.
</details>

Why did we go through this long explanation? First, to get more familiar with the concept of homogeneous coordinates. In CG we work with mostly two types of matrices: 4x4 affine transformation matrices and 4x4 projection matrices. Affine transformation matrices keep the transformed points w-coordinate equal to 1 as we just saw, but projection matrices, which are the matrices we will study in this lesson, don't. A point transformed by a projection matrix will thus require the x' y' and z' coordinates to be normalized, which as you know now isn't necessary when points are transformed by an affine transformation matrix.

|-table{Affine Transform Matrix, Projection Matrix (perspective or orthographic)}
|-row
|-cell
Input 3D point is implicitly converted to homogeneous coordinates {x, y, z, w = 1}
|-cell
Input 3D point is implicitly converted to homogeneous coordinates {x, y, z, w = 1}</td>
|-row
|-cell
\(m_{30}, m_{31}, m_{32}\) and \(m_{33}\) are always equal to {0,0,0,1} respectively.
|-cell
\(m_{30}, m_{31}, m_{32}\) and \(m_{33}\) take on values that are specific to projection matrices. We will soon explain what these values are.
|-row
|-cell
w' is always equal to 1.  
No need to compute w'.
$$
\begin{array}{l}
w' &=& x * (m_{30} = 0) + \\ 
&& y * (m_{31}= 0) + \\ 
&& z * (m_{32} = 0) + \\
&& (w = 1) * 1 \\
& = & 1
\end{array}
$$
|-cell
w' could be different than 1 and needs to be computed explicitly:
$$
\begin{array}{l}
w' & = & x * (m_{30} != 0) + \\
&& y * (m_{31} != 0) + \\
&& z * (m_{32} != 0) + \\
&& (w = 1) * (m_{33} != 1) \\
& != & 1
\end{array}
$$
|-row
|-cell
Normalization is never needed.
$$
\begin{array}{l}
P'_H = \{x', y', z', w'=1\} \\
P'_C = \{x', y', z'\}
\end{array}
$$
|-cell
Normalization is necessary if \(w'!=1\).
$$
\begin{array}{l}
P'_H = \{x', y', z', w'!=1\} \\
P'_C = \{x'/w', y'/w', z'/w'\}
\end{array}
$$
|-

Why does it matter to learn about the difference between affine transformation and projection matrices? It matters a lot. First, because if you multiply a point by a projection matrix, you will need to use a version of the point-matrix multiplication function that computes w explicitly and then normalize the coordinates of the transformed point. Something similar to this function (version 2):

```
template<typename S> 
void multVecMatrix(const Vec3<S> &src, Vec3<S> &dst) const 
{ 
    S a, b, c, w; 
 
    // since src.w is assumed to always be equal to 1, it needn't be defined explicitly
    a = src.x * x[0][0] + src.y * x[1][0] + src.z * x[2][0] + x[3][0]; 
    b = src.x * x[0][1] + src.y * x[1][1] + src.z * x[2][1] + x[3][1]; 
    c = src.x * x[0][2] + src.y * x[1][2] + src.z * x[2][2] + x[3][2]; 
    w = src.x * x[0][3] + src.y * x[1][3] + src.z * x[2][3] + x[3][3]; 
 
    dst.x = a / w; 
    dst.y = b / w; 
    dst.z = c / w; 
} 
```

If you don't use this function with a projection matrix, the result will be wrong. Of course, for optimization reasons, you don't want to use this function when you use the more common affine transformation matrices (w' doesn't need to be computed and the transformed coordinates don't need to be normalized). Thus, you will need to be careful about eventually creating two functions and calling one or the other depending on the type of matrix you are using. In practice though, this is rarely done. Programmers often don't bother and just use something like this (it at least avoids an unnecessary division when w is equal to 1):

```
template<typename S> 
void multVecMatrix(const Vec3<S> &src, Vec3<S> &dst) const 
{ 
    S a, b, c, w; 
 
    // since src.w is assumed to always be equal to 1, it needn't be defined explicitly
    a = src.x * x[0][0] + src.y * x[1][0] + src.z * x[2][0] + x[3][0]; 
    b = src.x * x[0][1] + src.y * x[1][1] + src.z * x[2][1] + x[3][1]; 
    c = src.x * x[0][2] + src.y * x[1][2] + src.z * x[2][2] + x[3][2]; 
    w = src.x * x[0][3] + src.y * x[1][3] + src.z * x[2][3] + x[3][3]; 
 
    if (w != 1) { 
        dst.x = a / w; 
        dst.y = b / w; 
        dst.z = c / w; 
    } 
    else { 
        dst.x = a; 
        dst.y = b; 
        dst.z = c; 
    } 
} 
```

This code can be used with both the standard affine transformation and projection matrices. Though the main reason we talked about homogeneous coordinates so much is that the normalization step plays a key role in the way projection matrices work as we will see in the next chapter.