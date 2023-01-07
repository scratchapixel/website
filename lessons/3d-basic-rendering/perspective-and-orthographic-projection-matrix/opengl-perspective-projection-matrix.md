## The OpenGL Perspective Projection Matrix

In all OpenGL books and references, the perspective projection matrix used in OpenGL is defined as:

$$
\left[\begin{array}{cccc}
{ \dfrac{2n}{ r-l } } &amp; 0 &amp; { \dfrac{r + l} { r-l } } &amp; 0 \\
0 &amp; { \dfrac{2n}{ t-b } } &amp; { \dfrac{t + b}{ t-b } } &amp; 0 \\
0 &amp; 0 &amp; -{\dfrac{f+n}{f-n}} &amp; -{\dfrac{2fn}{f-n}}\\
0 &amp; 0 &amp; -1&amp; 0\\
\end{array}\right]
$$

What similarities does this matrix have with the matrix we studied in the previous chapter? First, it is important to remember that matrices in OpenGL are defined using a column-major order (as opposed to row-major order). In the lesson on [Geometry](/lessons/mathematics-physics-for-computer-graphics/geometry/row-major-vs-column-major-vector), we have explained that to go from one order to the other we can simply transpose the matrix. If we were transposing the above matrix we would get:

$$
\left[\begin{array}{cccc}
{ \dfrac{2n}{ r-l } } &amp; 0 &amp; 0 &amp; 0 \\
0 &amp; { \dfrac{2n}{ t-b } } &amp; 0 &amp; 0 \\
{ \dfrac{r + l}{ r-l } } &amp; { \dfrac{t + b}{ t-b } } &amp; -{\dfrac{f+n}{f-n}} &amp; {\color{\red}{-1}}\\
0 &amp; 0 &amp; -{\dfrac{2fn}{f-n}}&amp; 0\\
\end{array}\right]
$$

This is the matrix we would be using on Scratchapixel (as we use row vectors). But in OpenGL, you would have to use the first matrix (as OpenGL uses column vectors by default though you can change that if you want in OpenGL4.x). Pay attention to the element in red (third row and fourth column). When we multiply a homogeneous point with this matrix, the point's w coordinate is multiplied by this element and the value of w ends up being the projected point's z coordinate:

$$
\left[\begin{array}{cccc}x' &amp; y' &amp; z' &amp; w'\end{array}\right] = \\
\left[\begin{array}{cccc}x &amp; y &amp; z &amp; w = 1\end{array}\right] * 
\left[\begin{array}{cccc} 
{ \dfrac{2n}{ r-l } } &amp; 0 &amp; 0 &amp; 0 \\ 
0 &amp; { \dfrac{2n}{ t-b } } &amp; 0 &amp; 0 \\ 
{ \dfrac{r + l}{ r-l } } &amp; { \dfrac{t + b}{ t-b } } &amp; -{\dfrac{f+n}{f-n}} &amp; {\color{\red}{-1}}\\ 
0 &amp; 0 &amp; -{\dfrac{2fn}{f-n}}&amp; 0\\ 
\end{array}\right]
$$

$$P'_w = 0 * P_x + 0 * P_x * -1 * P_z + 0 * 0 = -P_z.$$

## Principle

In summary, we already know that this matrix is set up properly for the z-divide. Now let's see how the points are projected in OpenGL. The principle is course the same as in the previous chapter. We draw a line from the camera's origin to the point P we want to project, and the intersection of this line with the image plane indicates the position of the projected point Ps. The setup is the same as in figure 1 from the previous chapter, however, note that in OpenGL the image plane is located on the near clipping plane (rather than being exactly one unit away from the camera's origin).

![Figure 1: we use the property of similar triangles to find the position of Ps.](/images/perspective-matrix/projectionOpenGL.png?)

The trick we used in chapter 1 with similar triangles can be used here again. The triangles \(\Delta ABC\) and \(\Delta DEF\) are similar. Therefore we can write:

$${\dfrac{AB}{DE}} = {\dfrac{BC}{EF}}.$$

If we replace AB with \(n\), the near clipping plane, DE with \(Pz\) (the z coordinate of P), and EF with \(Py\) (the y coordinate of P) we can re-write this equation as (equation 1):

$${\dfrac{n}{-P_z}} = {\dfrac{BC}{P_y}} \rightarrow BC = Ps_y = {\dfrac{n * P_y}{-P_z}}.$$

As you can see, the only difference with the equation from the previous chapter is the term \(n\) in the numerator but the principle of the division by \(Pz\) is the same (note that because the camera is oriented along the negative z-axis, \(Pz\) is negative: \(P_z \lt 0\). To keep the y-coordinate of the projected point positive, since \(Py\) is positive, we need to negate \(Pz\)). If we follow the same reasoning we find the x-coordinate of the projected point using the following equation (equation 2):

$$Ps_x =\dfrac{n * P_x}{-P_z}.$$

## Derivation

![Figure 2: the frustum or viewing volume of a camera is defined by the camera's field of view, the near and far clipping planes, and the image aspect ratio. In OpenGL, points are projected on the front face of the frustum (the near clipping plane)](/images/perspective-matrix/projectionOpenGL2.png?)

Now that we have two values for \(Ps_x\) and \(Ps_y\) we still need to explain how they relate to the OpenGL perspective matrix. The goal of a projection matrix is to remap the values projected onto the image plane to a unit cube (a cube whose minimum and maximum extents are (-1,-1,-1) and (1,1,1) respectively). However, once the point P is projected on the image plane, Ps is visible if its x- and y- coordinates are contained within the range [left, right] for x and [bottom, top] for y. This is illustrated in Figure 2. We have already explained in the lesson [3D Viewing: the Pinhole Camera Model](/lessons/3d-basic-rendering/3d-viewing-pinhole-camera/implementing-virtual-pinhole-camera) how these left, right, bottom top coordinates are computed but we will explain this again in this chapter. These screen coordinates define the limits or boundaries on the image plane of the visible points (all the points contained in the viewing frustum and projected on the image plane). If we assume that \(Ps_x\) is visible, then we can write:

$$l \leq Ps_x \leq r.$$

where \(l\) and \(r\) and the left and right coordinates respectively. Our goal now is to remap the term in the middle (\(Ps_x\)) such that the final value lies in the range [-1,1] (the dimension of the unit cube along the x-axis). We have already introduced these equations in the [previous lesson](/lessons/3d-basic-rendering/rasterization-practical-implementation/projection-stage) but we will write them down one more time. Let's start by removing \(l\) from all the terms and re-write the above equation as:

$$0 \leq Ps_x - l \leq r - l.$$

We can normalize the term on the right by dividing all the terms of this formula by \(r-l\):

$$0 \leq {\dfrac{Ps_x - l}{r-l}} \leq 1.$$

Then we will multiply all the terms by 2:

$$0 \leq 2{\dfrac{Ps_x - l}{r-l}} \leq 2.$$

If we remove -1 from all the terms:

$$-1 \leq 2{\dfrac{Ps_x - l}{r-l}} -1 \leq 1.$$

We now have the central term remapped to the range [-1,1] which is what we wanted though we can keep arranging the terms even further:

$$-1 \leq 2{ \dfrac{Ps_x - l}{r-l} } - { \dfrac{r-l}{r-l} } \leq 1.$$

If we develop we get:

$$-1 \leq { \dfrac{2Ps_x - 2l - r + l}{r-l} } \leq 1.$$

Therefore:

$$-1 \leq { \dfrac{2Ps_x - l - r}{r-l} } \leq 1 \rightarrow -1 \leq { \dfrac{2Ps_x}{r-l} } - { \dfrac{r + l}{r - l} } \leq 1.$$

These two terms are quite similar to the first two terms of the first row in the OpenGL perspective projection matrix. We are getting closer. If we replace \(Ps_x\) from the previous equation with equation 2 we get:

$$-1 \leq { \dfrac{2 n P_x}{-P_z{(r-l)}} } - { \dfrac{r + l}{r - l} } \leq 1.$$

We can very easily encode this equation using the matrix form. If we replace the first and third coefficients of the matrix first row with the first and second term of this formula here is what we get:

$$
\left[\begin{array}{cccc}
{ \dfrac{2n}{ r-l } } &amp; 0 &amp; { \dfrac{r + l}{ r-l } } &amp; 0 \\
... &amp; ... &amp; ... &amp; ... \\
... &amp; ... &amp; ... &amp; ... \\
0 &amp; 0 &amp; -1&amp; 0\\
\end{array}\right]
$$

Remember that the OpenGL matrix uses column-major ordering therefore we will have to write the multiplication sign to the right of the matrix and the point coordinates using a column vector:

$$
\left[\begin{array}{cccc}
{ \dfrac{2n}{ r-l } } &amp; 0 &amp; { \dfrac{r + l}{ r-l } } &amp; 0 \\
... &amp; ... &amp; ... &amp; ... \\
... &amp; ... &amp; ... &amp; ... \\
0 &amp; 0 &amp; -1&amp; 0\\
\end{array}\right] * \left[ \begin{array}{cccc}x \\ y \\ z \\ w\end{array}\right]
$$

Computing \(Ps_x\) using this matrix gives:

$$Ps_x = { \dfrac{2n}{ r-l } } P_x + 0 * P_y + { \dfrac{r + l}{ r-l } } * P_z + 0 * P_w.$$

<details>
You should be familiar with the concept of matrix-vector multiplication at this point as well as the concept of row vs. column-major vectors and matrices. In short because in this particular example, we use a column-major vector notation (that's the convention used by OpenGL not by Scratchapixel - we prefer the row-major notation) to compute the transformed coordinate of the first coordinate (x) you need to use the coefficient of the matrix first row and the vector's coordinates in the following way: $Px_{transform} = M_{00} * Px + M_{01} * Py + M_{02} * Pz + M_{03} * Pw.$ If you are not familiar with these concepts read the lesson on [Geometry](/lessons/mathematics-physics-for-computer-graphics/geometry/).
</details>

And since \(Ps_x\) will be divided at the end of the process by \(-P_z \) when we will convert Ps from homogeneous to cartesian coordinates, we get:

$$
Ps_x = \dfrac { \dfrac {2n} { r-l } P_x } { -P_z} + \dfrac{ \dfrac {r + l} { r-l } P_z } { -P_z} \rightarrow \dfrac {2n P_x} { -P_z (r-l) } - \dfrac {r + l} { r-l }.
$$

This is the first coordinate of the projected point Ps computed using the OpenGL perspective matrix. The derivation is quite long and we will skip it for \(Ps_y\). However, if you follow the steps we have been using for \(Ps_x\) doing it yourself shouldn't be a problem. You just need to replace \(l\) and \(r\) with \(b\) and \(t\) and you end up with the following formula:

$$-1 \leq { \dfrac{2 n P_y}{-P_z{(t-b)}} } - { \dfrac{t + b}{t - b} } \leq 1.$$

We can get this result with a point-matrix multiplication if we replace the second and third coefficients of the matrix second row with the first and second term of this equation:

$$
\left[\begin{array}{cccc}
{ \dfrac{2n}{ r-l } } &amp; 0 &amp; { \dfrac{r + l}{ r-l } } &amp; 0 \\
0 &amp; { \dfrac{2n}{ t-b } } &amp; { \dfrac{t + b}{ t-b } } &amp; 0 \\
... &amp; ... &amp; ... &amp; ... \\
0 &amp; 0 &amp; -1&amp; 0\\
\end{array}\right]
$$

Computing \(Ps_y\) using this matrix gives:

$$Ps_y = 0 * P_x + { \dfrac{2n}{ (t-b) } } * P_y + { \dfrac{t + b}{ t-b } } * P_z + 0 * P_w$$

and after the division by \(-P_z\):

$$Ps_y = \dfrac { \dfrac {2n} {t - b} P_y } { -P_z} + \dfrac{ \dfrac {t + b} {t - b} P_z } { -P_z} \rightarrow \dfrac {2n P_y} { -P_z (t - b) } - \dfrac {t + b} {t - b}$$

Our matrix works again. All we are left to do to complete it is find a way of remapping the z-coordinate of the projected points to the range [-1,1]. We know that the x- and y-coordinates of P don't contribute to the calculation of the projected point z-coordinate. Thus the first and second coefficients of the matrix third row which would be multiplied by P x- and y-coordinates, are necessarily zero (in green). We are left with two coefficients A and B in the matrix which is unknown (in red).

$$
\left[\begin{array}{cccc}
{ \dfrac{2n}{ r-l } } &amp; 0 &amp; { \dfrac{r + l}{ r-l } } &amp; 0 \\
0 &amp; { \dfrac{2n}{ t-b } } &amp; { \dfrac{t + b}{ t-b } } &amp; 0 \\
{ \color{\green}{ 0 } } &amp; { \color{\green}{ 0 } } &amp;  { \color{\red}{ A } } &amp;{ \color{\red}{ B } }\\
0 &amp; 0 &amp; -1 &amp; 0 \\
\end{array}\right]
$$

If we write the equation to compute \(Ps_z\) using this matrix, we get (remember that \(Ps_z\) is also divided by \(Ps_w\) when the point is converted from homogeneous to cartesian coordinates, and that \(P_w = 1\)):

$$
Ps_z = \dfrac{0 * P_x + 0 * P_y + A * P_z + B * P_w}{Ps_w = -P_z} \rightarrow \dfrac{A P_z + B}{Ps_w = -P_z}.
$$

We need to find the value of A and B. Hopefully we know that when \(P_z\) lies on the near clipping plane, \(Ps_z\) needs to be remapped to -1 and when \(P_z\) lies on the far clipping plane, \(Ps_z\) needs to be remapped to 1. Therefore, we need to replace \(Ps_z\) with \(n\) and \(f\) in the equation to get two new equations (note that the z-coordinate of all the points projected on the image plane is negative but \(n\) and \(f\) are positive therefore we will use \(-n\) and \(-f\) instead):

$$
\left\{ \begin{array}{ll} \dfrac{(P_z=-n)A + B}{(-P_z=-(-n)=n)} = -1 &amp;\text{ when } P_z = n\\ \dfrac{(P_z=-f)A + B}{(P_z=-(-f)=f)} = 1 &amp; \text{ when } P_z = f \end{array} \right. \\ \rightarrow  \left\{ \begin{array}{ll} {-nA + B} = -n &amp; (1)\\  {-fA + B} = f &amp; (2) \end{array} \right.
$$

Let's solve for B in equation 1:

$$B = -n + An.$$

And substitute B in equation 2 with this equation:

$$-fA - n + An = f.$$

Then solve for A:

$$-fA + An = f + n \rightarrow -(f - n)A = f + n \rightarrow A = -\dfrac{f + n}{f - n}.$$

Now that we have a solution for A, it is easy to find B. We just replace A in equation 1 to find B:

$$B = -n + An= -n -\dfrac{f + n}{f - n} n = \\-(1+\dfrac{f + n}{f - n}) n = - \dfrac{{(f -n + f + n)}n}{f - n}=-\dfrac { 2fn }{f -n}.$$

We can replace the solution we found for A and B in our matrix and we finally get:

$$\left[\begin{array}{cccc} { \dfrac{2n}{ r-l } } &amp; 0 &amp; { \dfrac{r + l}{ r-l } } &amp; 0 \\ 0 &amp; { \dfrac{2n}{ t-b } } &amp; { \dfrac{t + b}{ t-b } } &amp; 0 \\ 0 &amp; 0 &amp; -{\dfrac{f+n}{f-n}} &amp; -{\dfrac{2fn}{f-n}}\\ 0 &amp; 0 &amp; -1&amp; 0\\ \end{array}\right]$$

which is the OpenGL perspective projection matrix.

<details>
Note that in the previous chapter we chose to remap z to the range [0,1]. You can technically remap this to whatever you want but [0,1] is also a common choice. Finding the equation for A and B just require a simple change: $ \left\{ \begin{array}{ll} \dfrac{(P_z=-n)A + B}{(-P_z=-(-n)=n)} = 0 &\text{ when } P_z = n\\ \dfrac{(P_z=-f)A + B}{(P_z=-(-f)=f)} = 1 & \text{ when } P_z = f \end{array} \right. \\ \rightarrow \left\{ \begin{array}{ll} {-nA + B} = 0 & (1)\\ {-fA + B} = f & (2) \end{array} \right. $ From which we can derive that: $A = -\dfrac{f}{(f-n)}.$ And: $B = -\dfrac{fn}{(f-n)}.$ Which are the equations we used in the previous chapter.
</details>

![Figure 3: the remapping of the projected point's z coordinate is nonlinear. This graph shows the result of \(\scriptsize Ps_z\) for near = 1 and far = 5.](/images/perspective-matrix/depth2.png?)

The remapping of the z-coordinate has the property of representing points nearer to the camera with more numerical precision than the points further away. As we explained in the previous chapter (in the notes), this property can be a problem when the lack of numerical precision causes some adjacent samples to have the same depth value after they have been projected to the screen, when their z coordinates in world space are different, a problem known as **z-fighting**. The issue can't be solved (we are always limited to the precision that can be stored in a single-precision floating-point number though the problem can be minimized if the near and far clipping planes fit respectively as closely as possible to the nearest and furthest object visible in the scene. This is the reason why adjusting the clipping planes is always recommended.

![](/images/perspective-matrix/optimized-clip-planes.png?)

## The field of view and Image Aspect Ratio

![Figure 4: side view of the camera. The triangle ACD's apex defines the camera's vertical field of view (FOV). The image plane location is defined by the near-clipping plane distance. From these two values (the FOV and the near clipping plane) we can compute the top coordinate using simple trigonometry](/images/perspective-matrix/openglsetup.png?)

You may have noticed that so far we haven't made any reference to the camera's field of view and image aspect ratio. However, we have said in the previous chapter and the lesson on cameras (in the basic section), that changing the FOV changes the extent of the scene we see through the camera. Thus the field of view and the image aspect ratio should be related to the projection process somehow. We deliberately ignored this detail until now to stay focused on the OpenGL perspective projection matrix which doesn't directly rely on the camera's field of view. But it does indirectly. The construction of the matrix depends on six parameters, the left, right, bottom, and top coordinates as well as the near and far clipping plane. The near and far clipping plane values are given by the user, but what about the left, right, bottom, and top coordinates. What are they, where do they come from and how do we calculate them? If you watch Figures 2 and 5 you can see that these coordinates correspond to the lower-left and upper-right corners of the frustum front face, the face on which the image of the 3D scene is projected. How do we compute these coordinates. Figure 4 represents a view of the camera from the side. What we want is to compute a value for the top coordinate, which is equal to the right-angle \(\scriptsize \Delta ABC\) triangle's opposite side. The angle subtended by AB and AC is half the field of view and the adjacent side of the triangle is the value for the near-clipping plane. Using trigonometry we can write:

$$\tan( \dfrac{ FOVY } {2}) = \dfrac{ opposite } { adjacent } = \dfrac {BC}{AB} = \dfrac{top}{near}$$

therefore:

$$top = \tan( \dfrac{ FOVY } {2}) * near $$

And since the bottom half of the camera is symmetrical to the upper half, we can write that:

$$bottom = -top$$

<details>
The angle of view can either be defined vertically or horizontally. OpenGL tends to define the field-of-view as being vertical (hence the Y in FOVY) but on Scratchapixel we use a horizontal angle-of-view (same as Maya and RenderMan).
</details>

![Figure 5: the image can be square (left) or rectangular (right). Note that the bottom-left coordinates and the top-right coordinates are symmetric about the x- and y-axis.](/images/perspective-matrix/openglsetup2.png?)

If you look at figure 5 though, two cases should be taken into consideration. The image can either be square or rectangular. In the case where the camera is square, it is straightforward to see that the left and bottom coordinates are the same, that the right and the top coordinates are also the same, and finally that if you mirror the bottom-left coordinates around the x- and y-axis, you get the top-right coordinates. Therefore if we can compute the top coordinates we can easily set the other three other coordinates:

$$
\begin{array}{l}
top = \tan( \dfrac{ FOV } {2}) * near\\
right = top\\
left = bottom = -top \end{array}
$$

If the camera is not square as with the frustum on the right inside of figure 4, computing the coordinates is slightly more complicated. The bottom and top coordinates are still the same but the left and right coordinates are scaled by the ratio defined as the image width over the image height, what we usually call the image aspect ratio. The final and general formulas for computing the left, right, and bottom coordinates are:

$$
\begin{array}{l}
aspect\;ratio = \dfrac{width}{height}\\
top = \tan( \dfrac{ FOV } {2}) * near\\
bottom = -top \\
right = top * aspect\;ratio\\
left = bottom = -top * aspect\;ratio
\end{array}
$$

The camera's field of view and image aspect ratio is used to calculate the left, right, bottom, and top coordinates which are themselves used in the construction of the perspective projection matrix. This is how they indirectly contribute to modifying how much of the scene we see through the camera.

## Test Program

To test the OpenGL perspective projection matrix we will re-use the code from the previous chapter. In the old fixed-function rendering pipeline, two functions were used to set the screen coordinates and the projection matrix. These functions were called `gluPerspective` (it was part of the `glue` library) and `glFrustum`. Do not use these functions anymore as they are deprecated (since OpenGL 3.1) in the new programmable rendering pipeline, though we are using them here in this lesson to show how they would have been implemented based on what we learned in this chapter (and you can still use them if you want in your CPU program to emulate them).

Setting up the perspective projection matrix in OpenGL was done through a call to glFrustum. The function took six arguments:

```
glFrustum(float left, float right, float bottom, float top, float near, float far);
```

The implementation of this function can be found in the code below (line 20). The function gluPerspective was used to set the screen coordinates. It took as arguments, the angle-of-view, the image aspect ratio (the image width over the image height), and the clipping planes.

```
void gluPerspective(float fovy, float spect, float zNear, float zFar);
```

In OpenGL, the angle-of-view is defined as the vertical angle-of-view (hence the y in the variable name). On Scratchapixel, we use the horizontal angle of view. An implementation of this function can be found in the code below (line 8). The rest of the code is the same. We first compute the screen coordinates, then the projection matrix. Then we loop over all the vertices of the teapot geometry, transform them from object/world space to camera space, and finally project them onto the screen using the perspective projection matrix. Remember that the matrix remaps the projected point to NDC space. Thus as in the previous version of the code, visible points are contained within the range [-1,1] in height and [-imageAspectRatio, imageAspectRatio] (which is [-1,1] if the image is square) in width.

```
#include <cstdio> 
#include <cstdlib> 
#include <fstream> 
#include "geometry.h" 
#include "vertexdata.h" 
 
// compute screen coordinates first
void gluPerspective( 
    const float &angleOfView, 
    const float &imageAspectRatio, 
    const float &n, const float &f, 
    float &b, float &t, float &l, float &r) 
{ 
    float scale = tan(angleOfView * 0.5 * M_PI / 180) * n; 
    r = imageAspectRatio * scale, l = -r; 
    t = scale, b = -t; 
} 
 
// set the OpenGL perspective projection matrix
void glFrustum( 
    const float &b, const float &t, const float &l, const float &r, 
    const float &n, const float &f, 
    Matrix44f &M) 
{ 
    // set OpenGL perspective projection matrix
    M[0][0] = 2 * n / (r - l); 
    M[0][1] = 0; 
    M[0][2] = 0; 
    M[0][3] = 0; 
 
    M[1][0] = 0; 
    M[1][1] = 2 * n / (t - b); 
    M[1][2] = 0; 
    M[1][3] = 0; 
 
    M[2][0] = (r + l) / (r - l); 
    M[2][1] = (t + b) / (t - b); 
    M[2][2] = -(f + n) / (f - n); 
    M[2][3] = -1; 
 
    M[3][0] = 0; 
    M[3][1] = 0; 
    M[3][2] = -2 * f * n / (f - n); 
    M[3][3] = 0; 
} 
 
void multPointMatrix(const Vec3f &in, Vec3f &out, const Matrix44f &M) 
{ 
    //out = in * Mproj;
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
    float imageAspectRatio = imageWidth / (float)imageHeight; 
    float b, t, l, r; 
    gluPerspective(angleOfView, imageAspectRatio, near, far, b, t, l, r); 
    glFrustum(b, t, l, r, near, far, Mproj); 
    unsigned char *buffer = new unsigned char[imageWidth * imageHeight]; 
    memset(buffer, 0x0, imageWidth * imageHeight); 
    for (uint32_t i = 0; i < numVertices; ++i) { 
        Vec3f vertCamera, projectedVert; 
        multPointMatrix(vertices[i], vertCamera, worldToCamera); 
        multPointMatrix(vertCamera, projectedVert, Mproj); 
        if (projectedVert.x < -imageAspectRatio || projectedVert.x > imageAspectRatio || projectedVert.y < -1 || projectedVert.y > 1) continue; 
        // convert to raster space and mark the position of the vertex in the image with a simple dot
        uint32_t x = std::min(imageWidth - 1, (uint32_t)((projectedVert.x + 1) * 0.5 * imageWidth)); 
        uint32_t y = std::min(imageHeight - 1, (uint32_t)((1 - (projectedVert.y + 1) * 0.5) * imageHeight)); 
        buffer[y * imageWidth + x] = 255; 
        //std::cerr << "here sometmes" << std::endl;
    } 
    // export to image
    std::ofstream ofs; 
    ofs.open("./out.ppm"); 
    ofs << "P5\n" << imageWidth << " " << imageHeight << "\n255\n"; 
    ofs.write((char*)buffer, imageWidth * imageHeight); 
    ofs.close(); 
    delete [] buffer; 
 
    return 0; 
} 
```

We mentioned in the first chapter, that even if matrices are built differently (and look different), they should still always give the same result (a point in 3D space should always be projected to the same pixel location on the image). If we compare the result of projecting the teapot's vertices using the first matrix with the result of projecting the same vertices with the same camera settings (same field of view, image aspect ratio, near and far clipping planes) and the OpenGL perspective projection matrix, we get the same two images (see image below).

![](/images/perspective-matrix/perspprojresults.png?)

The source code of this program is available in the source code chapter of this lesson.