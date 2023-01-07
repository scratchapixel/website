## A Simple Perspective Matrix

**A word of warning again**. The matrix we will present in this chapter is different from the projection matrix that is being used in APIs such as OpenGL or Direct3D. Though, it technically produces the same results. In the lesson [3D Viewing: the Pinhole Camera Model](/lessons/3d-basic-rendering/3d-viewing-pinhole-camera/) we learned how to compute the screen coordinates (left, right, top and bottom) based on the camera near clipping plane and angle-of-view (in fact, we learned how to compute these coordinates based on the parameters of a physically based camera model). We then used these coordinates to decide if the projected points were visible or not in the image (they would only be visible if their coordinates were contained within the screen coordinates). In the lesson [Rasterization: a Practical Implementation](/lessons/3d-basic-rendering/rasterization-practical-implementation/projection-stage), we learned how to remap the projected point coordinates to NDC coordinates (coordinates in the range [-1,1]) using the screen coordinates. In other words, to avoid having to compare the projected point coordinates to the screen coordinates, we remapped the point coordinates first to the range [-1,1] using the screen coordinates. Deciding whether a point is visible or not is just a matter of testing if any of its coordinates is lower than -1 or greater than 1.

In this chapter, we will use a slightly different approach. We will assume that the screen coordinates are (-1,1) for the left and right coordinates and (-1,1) for the bottom and top coordinates (assuming a square screen) to start with (since this is the range we want to test the coordinates against), and we will account for the camera field-of-view by scaling the projected point coordinates directly (rather than using the screen coordinates scaled by the angle-of-view to remap the points coordinates to NDC space). Both methods have the same effect.

Recall from the lesson on Geometry that the multiplication of a point by a matrix is as follows:

$$
\begin{equation}
\begin{bmatrix} x &amp; y &amp; z &amp; w
\end{bmatrix} * 
\begin{bmatrix}
m_{00} &amp; m_{01} &amp; m_{02} &amp; m_{03}\\ 
m_{10} &amp; m_{11} &amp; m_{12} &amp; m_{13}\\
m_{20} &amp; m_{21} &amp; m_{22} &amp; m_{23}\\
m_{30} &amp; m_{31} &amp; m_{32} &amp; m_{33}
\end{bmatrix}
\end{equation}
$$

$$
\begin{array}{l}
x' = x * m_{00} + y * m_{10} + z * m_{20} + w * m_{30}\\
y' = x * m_{01} + y * m_{11} + z * m_{21} + w * m_{31}\\
z' = x * m_{02} + y * m_{12} + z * m_{22} + w * m_{32}\\
w' = x * m_{03} + y * m_{13} + z * m_{23} + w * m_{33}
\end{array}
$$

Also, remember from the previous chapter, that point P', i.e. the projection of P onto the image plane, can be computed by dividing the x- and y-coordinates of P by the **inverse** of the point z-coordinate:

$$
\begin{array}{l}
P'_x=\dfrac{P_x}{-P_z} \\
P'_y=\dfrac{P_y}{-P_z} \\
\end{array}
$$

How do we compute P' using a point-matrix multiplication?

![Figure 1: when you create a camera, it is by default aligned along the world coordinate system negative z-axis. This is a convention used by most 3D applications.](/images/perspective-matrix/camera.png?)

First, x', y' and z' (the coordinates of P') in the equation above need to be set with x, y, and -z respectively (where x,y and z are the coordinates of the point P we want to project). Why do we want to set z' to -z instead of just z? Remember that when we transform points from world space to camera space, all points defined in the camera coordinate system and located in front of the camera have a negative z-value. This is because, by default, cameras always point down the negative z-axis (figure 1). We will also assign z to z' but invert its sign so that z' is positive:

$$
\begin{array}{l}
x' = x,\\
y' = y\\
z' = -z \:\:\: z' > 0\\
\end{array}
$$

If somehow within the point-matrix multiplication process, we could manage to divide x', y' and z' by -z, then we would end up with:

$$
\begin{array}{l}
x' = \dfrac {x}{-z},\\
y' = \dfrac {y}{-z}\\
z' = \dfrac {-z}{-z} = 1\\
\end{array}
$$

Which, as we know, are the equations to compute the projected point P' coordinates (don't worry too much about z' for now). Thus, again the question is, is it possible to get the same result with a point-matrix multiplication? If so, what would that matrix look like? Let's consider the problem step by step. First, we said we needed to set the coordinates x', y', and z' with the coordinates x, y, and -z respectively. This is simple. A simple identity matrix (with a slight modification) will do the trick:

$$
\begin{equation}
\begin{bmatrix} x &amp; y &amp; z &amp; (w=1)
\end{bmatrix} * 
\begin{bmatrix}
1 & 0 & 0 & 0\\ 
0 & 1 & 0 & 0\\ 
0 & 0 & -1 & 0\\ 
0 & 0 & 0 & 1\\ 
\end{bmatrix}
\end{equation}
$$

$$
\begin{array}{l}
x' = x * 1 + y * 0 + z * 0 + w * 0 &=&x\\
y' = x * 0 + y * 1 + z * 0 + w * 0 &=&y\\
z' = x * 0 + y * 0 + z * -1 + w * 0 &=&-z\\
w' = x * 0 + y * 0 + z * 0 + (w=1) * 1 &=&1\\
\end{array}
$$

Note here that the point we multiply the matrix with, has homogeneous coordinates or at least is implicitly assumed to be a point with homogeneous coordinates and whose fourth coordinate, w, is set to 1. The second step requires dividing x' and y' by -z. Now, recall what we said in the previous chapter about points with homogeneous coordinates. Point P is a point with homogeneous coordinates, and its fourth coordinate, w, is equal to 1. This is the condition for making it possible to multiply 3D points which originally are 3D points with Cartesian coordinates, by 4x4 matrices. This doesn't mean though, that the point-matrix multiplication operation can't set the value of w' (the fourth coordinates of the transformed point P') to something different than 1 (we know w' is always equal to 1 when affine transformation matrices are used, but this doesn't have to be the case with other types matrices such as ... projection matrices of course). To convert the point with homogeneous coordinates back to a point with Cartesian coordinates, we need to divide x', y', and z' by w' as explained in the previous chapter:

> ... the homogeneous point [x, y, z, w] corresponds to the three-dimensional point [x/w, y/w, z/w].

This operation requires dividing x', y', z' by w', and guess what, if somehow w' was equal to -z, then we would exactly get what we are looking for: dividing x', y', and z' by -z.

!!!
**The trick is to use the conversion from homogeneous to Cartesian coordinate in the point-matrix multiplication process to perform the perspective divide (dividing x and y by z to compute the projected point coordinates x' and y'). This requires assigning -z to w'.**
!!!

The question now is: can we change our perspective projection matrix (which is just a slightly modified version of the identity matrix at this stage) so that the result of the point-matrix multiplication sets w' to -z? To answer this question, let's look again at the point-matrix multiplication but let's focus for now on the w' coordinate only:

$$
\begin{array}{l}
w' = x * m_{03} + y * m_{13} + z * m_{23} + w * m_{33}
\end{array}{}
$$

We know that the point P w-coordinate is equal to 1\. Thus the above equation becomes:

$$
\begin{array}{l}
w' = x * m_{03} + y * m_{13} + \color{red}{z * m_{23}} + 1 * m_{33}
\end{array}{}
$$

But this is not important. What's important, is to note that z which is multiplied by the matrix coefficient \(m_{23}\) (in red) is used in this equation. And z is exactly what we want w' to be set with or more exactly -z. It is trivial to note that if the matrix coefficient \(\color{red}{m_{23}}\) was set to -1 and all the other matrix coefficients involved in computing w' were set to 0 (\(m_{03}\), \(m_{13}\) and \(m_{33}\) respectively), then we would get:

$$w' = x * 0 + y * 0 + \color{red}{z * -1} + 1 * 0 = -z.$$

Which is exactly the result we are looking for. In conclusion, to set w' to -z, the coefficients \(m_{03}\), \(m_{13}\) \(\color{red}{m_{23}}\) and \(m_{33}\) of the perspective projection matrix need to be set to 0, 0, -1 and 0 respectively. If we make these changes to our previous matrix, here is what the perspective projection matrix now looks like:

$$
\left[ \begin{array}{rrrr}x & y & z & 1\end{array} \right] * 
\left[ \begin{array}{rrrr}
1 & 0 & 0 & 0\\ 
0 & 1 & 0 & 0\\
0 & 0 & -1 & \color{red}{-1}\\
0 & 0 & 0 & 0
\end{array} \right]
$$

<details>
Note the difference between this matrix and a standard affine transformation matrix. Remember that for the latter, the coefficients of the fourth column are always set to {0, 0, 0, 1}. 

$$
\begin{bmatrix}
\color{green}{m_{00}} & \color{green}{m_{01}} & \color{green}{m_{02}} & \color{blue}{0}\\
\color{green}{m_{10}} & \color{green}{m_{11}} & \color{green}{m_{12}} & \color{blue}{0}\\
\color{green}{m_{20}} & \color{green}{m_{21}} & \color{green}{m_{22}} & \color{blue}{0}\\
\color{red}{T_x} & \color{red}{T_y} & \color{red}{T_z} & \color{blue}{1}\\
\end{bmatrix}
$$

In the current form of our projection matrix, the coefficients of this column are now set to {0, 0, -1, 0}. 

$$
\begin{bmatrix}
\color{green}{m_{00}} & \color{green}{m_{01}} & \color{green}{m_{02}} & \color{blue}{0}\\
\color{green}{m_{10}} & \color{green}{m_{11}} & \color{green}{m_{12}} & \color{blue}{0}\\
\color{green}{m_{20}} & \color{green}{m_{21}} & \color{green}{m_{22}} & \color{blue}{-1}\\
\color{red}{T_x} & \color{red}{T_y} & \color{red}{T_z} & \color{blue}{0}\\
\end{bmatrix}
$$

This has for effect to set w' to -z. And if -z is different than 1, then the coefficient of the transformed points will need to be normalized. This is how or when more precisely the perspective divide is performed when a point is multiplied by a projection matrix. It is important to understand this idea.
</details>

When this matrix is used in a point-matrix multiplication, we get:

$$
\begin{array}{ll}
x' = x * 1 + y * 0 + z * 0 + 1 * 0 & = & x\\
y' = x * 0 + y * 1 + z * 0 + 1 * 0 & = & y\\
z' = x * 0 + y * 0 + z * -1 + 1 * 0 & = & -z\\
w' = x * 0 + y * 0 + z * -1 + 1 * & = & -z
\end{array}
$$

Then divide all coordinates by w' to set the point's homogeneous coordinates back to Cartesian coordinates:

$$
\begin{array}{ll}
x' = \dfrac{x'=x}{w'=-z},\\
y' = \dfrac{y'=y}{w'=-z},\\
z' = \dfrac{z'=-z}{w'=-z} = 1.
\end{array}
$$

This is exactly the result we were aiming for. At this point in the chapter, we have a simple perspective projection matrix that can be used to compute P'. However, we still need to account for two things. First, we need to remap z' to the range [0,1]. To do so, we will be using the camera near and far clipping planes. Finally, we need to take into account the camera angle of view. This parameter controls how much of the scene we see (remember that we aim to simulate a pinhole camera model which is defined by a near and far clipping plane as well as a field-of-view).

## Remapping the Z-Coordinate

Another goal of the perspective projection matrix is to normalize the z-coordinate of P, that is, to scale its value between 0 and 1. To do so, we will use the near and far clipping planes of the camera (you can find more information on clipping planes in the lesson [3D Viewing: the Pinhole Camera Model](/lessons/3d-basic-rendering/3d-viewing-pinhole-camera/)). To achieve this goal, we will set the coefficients of the matrix used to calculate z' to certain values: 

$$z' = x * m_{20} + y * m_{21} + z * \color{green}{m_{22}} + 1 * \color{red}{m_{23}}$$

We will change the third (in green) and fourth (in red) coefficients of the third column to fulfill two conditions: when P lies on the near clipping plane, z' is equal to 0 after the z-divide, and when z lies on the far clipping plane, z' is equal to 1 after the z-divide. This remapping operation is obtained by setting these coefficients to:

$$-\dfrac{f}{(f-n)},$$

and

$$-\dfrac{f*n}{(f-n)}$$

respectively, where \(n\) stands for the near-clipping plane and \(f\) for the far-clipping plane (you can find a derivation of this equation in the next chapter). To convince you that this works, let's look at the result of z' when P lies on the near and far clipping planes (\m_{20}\) and \(m_{21}\) are equal to 0):

$$
\dfrac{\dfrac{-(z'=z=-n)*f-f*n}{(f-n)}}{(w'=-1*z=n)}=
\dfrac{\dfrac{n*f-f*n}{(f-n)}}{(w'=-1*z=n)}=0
$$

$$
\dfrac{\dfrac{-(z'=z=-f)*f-f*n}{(f-n)}}{(w'=-1*z=f)}=
\dfrac{\dfrac{f*f-f*n}{(f-n)}}{(w'=-1*z=f)}=
$$

$$\dfrac{\dfrac{f*(f-n)}{(f-n)}}{(w'=-1*z=f)}=\dfrac{f}{f}=1$$

When z equals \(n\) (the near clipping plane) you can see in the first line of the equation that the numerator is equal to 0. Therefore the result of the equation is 0. In the second line, we have replaced z with \(f\), the far clipping plane. By rearranging the terms, we can see that the (f-n) terms cancel out, and we are left with f divided by itself, which equals 1.

<details>
Question from a reader: "You give the solution for remapping z to 0 to 1, but how did you come up with these formulas?". We will explain how to derive these formulas in the chapter devoted to the OpenGL perspective projection matrix.
</details>

Our modified perspective projection matrix that projects P to P' and remaps the z'-coordinate of P' from 0 to 1 now looks like this:

$$
\left[\begin{array}{cccc}
1 &amp; 0 &amp; 0 &amp; 0 \\
0 &amp; 1 &amp; 0 &amp; 0 \\
0 &amp; 0 &amp; -\dfrac{f}{(f-n)} &amp; -1\\
0 &amp; 0 &amp; -\dfrac{f*n}{(f-n)}&amp; 0\\
\end{array}\right]
$$

<details>
The remapping of the z-coordinate from 0 to 1 is not a linear process. In the image on the right, we have plotted the result of z' with the near and far clipping planes set to 1 and 20, respectively. As is evident, the curve is steep for values in the interval [1:3] and quite flat for values greater than 7. It means that the precision of z' is high in the proximity of the near-clipping plane and low as we get closer to the far-clipping planes. If the range [near:far] is too large, depth precision problems known as **z-fighting** can arise in depth-based hidden surface renderers. It is therefore important to make this interval as small as possible to minimize the depth buffer precision problem.
![](/images/perspective-matrix/depth.png?)
</details>

## Taking the Field-of-View into Account

<details>
In this chapter, we will assume that the screen is a square and that the distance between the screen and the eye is equal to 1. This is only to simplify the demonstration. You will provide a more generic solution in the next chapter.
</details>

All we need to do to get a basic perspective projection matrix working is to account for the **angle of view** or field-of-view (**FOV**) of the camera. We know that by changing the focal length of a zoom lens on a real camera, we can change how much we see of a scene (the extent of the scene). We want our CG camera to work in the same way.

![Figure 2: changing the focal makes it possible to see more or less of the scene we photograph. As can be seen in this illustration though, it normally changes the screen window.](/images/perspective-matrix/focal.png?)

The size of the projection window is [-1:1] in each dimension. In other words, a projected point is visible, if its x- and y-coordinates are within the range [-1:1]. Points whose projected coordinates are not contained in this range are invisible and are not drawn.

![Figure 3: The field-of-view or FOV controls how much of the scene is viewed.](/images/perspective-matrix/camsetup1.png?)

Note that in our system, the screen window maximum and minimum values do not change. They are always in the range [-1:1] regardless of the value used for the FOV (we assume that the screen is a square). When points coordinates are contained within the range [-1,1] we say that they are defined in NDC space.

Remember from chapter 1, that the goal of the perspective projection matrix, is to project a point onto the screen and remap their coordinates to the range [-1,1] (or to NDC space).

The distance to the screen window from the eye position does not change either (it is equal to 1). When the FOV changes, however, we have just shown that the screen window should accordingly become larger or smaller (see figures 2 and 5). How do we reconcile this contradiction? Since we want the screen window to be fixed, what we will change instead are the projected coordinates. We will scale them up or down and test them against the fixed borders of the screen window. Let's work through a few examples.

![Figure 4: To account for the field-of-view effect while keeping the size of the screen window the same (in the range [-1:1]), we need to scale the points up or down, depending on the FOV value.](/images/perspective-matrix/focal2.png?)

Imagine a point whose projected x-y coordinates are (1.2, 1.3). These coordinates are outside the range [-1:1], and the point is therefore not visible. If we scale them down by multiplying them by 0.7, the new, scaled coordinates of the point become (0.84, 0.91). This point is now visible since both coordinates are in the range [-1:1]. This action would correspond to the physical action of zooming out. Zooming out means decreasing the focal length on a zoom lens or increasing the FOV. For the opposite effect, multiply by a value greater than 1. For example, imagine a point whose projected coordinates are (-0.5, 0.3). If you multiply these numbers by 2.1, the new, scaled coordinates are (-1.05, 0.63). The y-coordinate is still contained within the range [-1:1], but now the x-coordinate is lower than -1 and thus too far to the left. The point which was originally visible becomes invisible after scaling. What happened? You zoomed in.

To scale the projected coordinates up or down, we will use the field of view of the camera. The field-of-view (or angle-of-view) intuitively controls how much of the scene is visible to the camera. See the lesson [3D Viewing: the Pinhole Camera Model](/lessons/3d-basic-rendering/3d-viewing-pinhole-camera/) for more information.

<details>
The FOV can be either the horizontal or vertical angle. If the screen window is square, the choice of FOV does not matter, as all the angles are the same. If the frame aspect ratio is different than 1, however, the choice of FOV (check the lesson on cameras in the basic section). In OpenGL (GLUT more precisely), the FOV corresponds to the vertical angle. In this lesson, the FOV is considered to be the **horizontal angle** (which is also the case in Maya).
</details>

![Figure 5: Zooming in or out normally changes the size of the screen window. See how it becomes bigger or smaller as the field-of-view increases or decreases.](/images/perspective-matrix/fov.gif?)

The value of FOV, however, is not directly used; the tangent of the angle is used instead. In the CG literature, the FOV can be defined as either the angle or half of the angle that is subtended by the viewing cone. We believe it is more intuitive to see the FOV as the angular extent of the visible scene rather than as half of this angle (as represented in figures 3 and 5). To find a value that can be used to scale the projected coordinates, however, we need to divide the FOV angle by two. This explains why the FOV is sometimes expressed as the half-angle. Why do we divide the angle in half? What is of interest to us is the right triangle inscribed in the cone. The change in this angle between the hypothenuse and the adjacent side of the triangle (or the FOV half-angle) controls the length of the triangle's opposite side. By increasing or decreasing this angle, we can scale up or down the border of the image window. And since we need a value that is centered around 1, we will take the tangent of this angle to scale our projected coordinates. Note that when the FOV half-angle is 45 degrees (FOV is then 90 degrees), the tangent of this angle is equal to 1. Therefore, when we multiply the projected coordinates by 1, the coordinates do not change. For values of the FOV lesser than 90 degrees, the tangent of the half-angle gives values smaller than 1, and for values greater than 90 degrees, it gives values greater than 1. But the opposite effect is needed. Recall that zooming in should correspond to a decrease in FOV, so we need to multiply the projected point coordinates by a value greater than 1. To zoom out means that the FOV increases, so we need to multiply these coordinates by a value less than 1. Thus, we will use the reciprocal of the tangent or in other words, one over the tangent of the FOV half-angle.

Here is the final equation to compute the value used to scale the coordinates of the projected point:

$$S = \dfrac{1}{\tan(\dfrac{fov}{2}*\dfrac{\pi}{180})}$$

And thus we have the final version of our basic perspective projection matrix:

$$
\left[\begin{array}{cccc}
S &amp; 0 &amp; 0 &amp; 0 \\
0 &amp; S &amp; 0 &amp; 0 \\
0 &amp; 0 &amp; -\dfrac{f}{(f-n)} &amp; -1\\
0 &amp; 0 &amp; -\dfrac{f*n}{(f-n)}&amp; 0\\
\end{array}\right]
$$

## Are There Different Ways of Building this Matrix?

Yes and no. Some renderers may have a different implementation of the perspective projection matrix. This is the case with OpenGL. OpenGL used a function called `glFrustum` to create perspective projection matrices. This call takes as arguments, the left, right, bottom, and top coordinates in addition to the near and far clipping planes. Unlike our system, OpenGL assumes that the points in the scene are projected on the near-clipping planes, rather than on a plane that lies one unit away from the camera position. The matrix itself might also look slightly different. Be careful about the convention used for vectors and matrices. The projected point can be represented as either a row or column vector. Check also whether the renderer uses a left- or right-handed coordinate system, as that could change the sign of the matrix coefficients. Despite these differences, the underlying principle of the perspective projection matrix is the same for all renderers. They always divide the x- and y- coordinates of the point by its z-coordinate. In the end, all matrices should project the same points to the same pixel coordinates, regardless of the conventions or the matrix that is being used. We will study the construction of the OpenGL matrix in the next chapter.

## Test Program

To test our basic perspective projection matrix, we wrote a small program to project the vertices of a polygonal object (Newell's teapot) onto the image plane using the projection matrix we developed in this chapter. The program itself is simple in its implementation. A function is used to build the perspective projection matrix. Its arguments are the camera's near and far clipping plane, as well as the camera field-of-view defined in degrees. The vertices of the teapot are stored in an array (line 5). Each point is then projected onto the image plane using a simple point-matrix multiplication (line 51). Note that we first transform the points from world or object space to camera space. The function `multPointMatrix` computes the product of a point with a matrix. Note how we create the fourth component, w (line 25), and divide the result of the new point's coordinates by w, only if w is different than 1 (line 28). **This is where and when the z or perspective divide occurs**. A point is only visible if its projected x- and y- coordinates are contained within the interval [-1:1] (regardless of the image aspect ratio). Otherwise, the point is outside the boundaries of the camera's screen boundaries. If the point is contained within this interval, we need to remap these coordinates to raster space, i.e. pixel coordinates. This operation is simple. We remap the coordinates from [-1:1] to [0:1], multiply by the image size, and round the resulting floating digit to the nearest integer, as pixel coordinates must be integers.

```
#include <cstdio> 
#include <cstdlib> 
#include <fstream> 
#include "geometry.h" 
#include "vertexdata.h" 
 
void setProjectionMatrix(const float &angleOfView, const float &near, const float &far, Matrix44f &M) 
{ 
    // set the basic projection matrix
    float scale = 1 / tan(angleOfView * 0.5 * M_PI / 180); 
    M[0][0] = scale;  //scale the x coordinates of the projected point 
    M[1][1] = scale;  //scale the y coordinates of the projected point 
    M[2][2] = -far / (far - near);  //used to remap z to [0,1] 
    M[3][2] = -far * near / (far - near);  //used to remap z [0,1] 
    M[2][3] = -1;  //set w = -z 
    M[3][3] = 0; 
} 
 
void multPointMatrix(const Vec3f &in, Vec3f &out, const Matrix44f &M) 
{ 
    //out = in * M;
    out.x   = in.x * M[0][0] + in.y * M[1][0] + in.z * M[2][0] + /* in.z = 1 */ M[3][0]; 
    out.y   = in.x * M[0][1] + in.y * M[1][1] + in.z * M[2][1] + /* in.z = 1 */ M[3][1]; 
    out.z   = in.x * M[0][2] + in.y * M[1][2] + in.z * M[2][2] + /* in.z = 1 */ M[3][2]; 
    float w = in.x * M[0][3] + in.y * M[1][3] + in.z * M[2][3] + /* in.z = 1 */ M[3][3]; 
 
    // normalize if w is different than 1 (convert from homogeneous to Cartesian coordinates)
    if (w != 1) { 
        out.x /= w; 
        out.y /= w; 
        out.z /= w; 
    } 
} 
 
int main(int argc, char **argv) 
{ 
    uint32_t imageWidth = 512, imageHeight = 512; 
    Matrix44f Mproj; 
    Matrix44f worldToCamera; 
    worldToCamera[3][1] = -10; 
    worldToCamera[3][2] = -20; 
    float angleOfView = 90; 
    float near = 0.1; 
    float far = 100; 
    setProjectionMatrix(angleOfView, near, far, Mproj); 
    unsigned char *buffer = new unsigned char[imageWidth * imageHeight]; 
    memset(buffer, 0x0, imageWidth * imageHeight); 
    for (uint32_t i = 0; i < numVertices; ++i) { 
        Vec3f vertCamera, projectedVert; 
        multPointMatrix(vertices[i], vertCamera, worldToCamera); 
        multPointMatrix(vertCamera, projectedVert, Mproj); 
        if (projectedVert.x < -1 || projectedVert.x > 1 || projectedVert.y < -1 || projectedVert.y > 1) continue; 
        // convert to raster space and mark the position of the vertex in the image with a simple dot
        uint32_t x = std::min(imageWidth - 1, (uint32_t)((projectedVert.x + 1) * 0.5 * imageWidth)); 
        uint32_t y = std::min(imageHeight - 1, (uint32_t)((1 - (projectedVert.y + 1) * 0.5) * imageHeight)); 
        buffer[y * imageWidth + x] = 255; 
    } 
    // save to file
    std::ofstream ofs; 
    ofs.open("./out.ppm"); 
    ofs << "P5\n" << imageWidth << " " << imageHeight << "\n255\n"; 
    ofs.write((char*)buffer, imageWidth * imageHeight); 
    ofs.close(); 
    delete [] buffer; 
 
    return 0; 
} 
```

To test our program, we have rendered an image of the teapot in a commercial renderer using the same camera settings and combined it with the image produced by our code. They match, as expected (the teapot geometry and the files of this program can be found in the Source Code chapter at the end of this lesson).

![](/images/perspective-matrix/teapot-projmat.png?)

## What's Next?

In the next chapter, we will learn how to construct the perspective projection matrix used in OpenGL. The principles are the same, but instead of mapping the points to an image plane one unit from the camera position, it projects the point onto the near clipping plane and it remaps the projected point coordinates to NDC space using the screen coordinates which are themselves computed from the camera near clipping plane and angle-of-view. This results in a different matrix. We will then learn about the **orthographic projection** matrix.