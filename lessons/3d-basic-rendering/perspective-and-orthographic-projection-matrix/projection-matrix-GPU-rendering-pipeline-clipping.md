## What Will We Study in this Chapter?

In the first chapter of this lesson, we said that projection matrices were used in the GPU rendering pipeline. We mentioned that there were two GPU rendering pipelines: the old one, called the fixed-function pipeline, and the new one which is generally referred to as the programmable rendering pipeline. We also talked about how clipping, the process that consists of discarding or trimming primitives that are either outside or straddling across the boundaries of the frustum, happens somehow while the points are being transformed by the projection matrix. Finally, we also explained that in fact, projection matrices don't convert points from camera space to NDC space but to **homogeneous clip space**. It is time to provide more information about these different topics. Let's explain what it means when we say that clipping happens while the points are being transformed. Let's explain what clip space is. And finally, let's review how the projection matrices are used in the old and the new GPU rendering pipeline.

## Clipping and Clip Space

![Figure 1: example of clipping in 2D. At the clipping stage, new triangles may be generated wherever the original geometry overlaps the boundaries of the viewing frustum.](/images/perspective-matrix/clipping2.png?)

![Figure 2: example of clipping in 3D.](/images/perspective-matrix/clipping6.png?)

Let's recall quickly that the main purpose of clipping is to essentially "reject" geometric primitives which are behind the eye or located exactly at the eye position (this would mean a division by 0 which we don't want) and more generally trim off part of the geometric primitives which are outside the viewing area (more information on this topic can be found in chapter 2). This viewing area is defined by the truncated pyramid of the perspective or viewing frustum. Any professional rendering system somehow needs to implement this step. Note that the process can result in creating more triangles as shown in figure 1 than the scenes initially contained.

The most common clipping algorithms are the Cohen-Sutherland algorithm for lines and the Sutherland-Hodgman algorithm for polygons. It happens that clipping is more easily done in clip space than in camera space (before vertices are transformed by the projection matrix) or screen space (after the perspective divide). Remember that when the points are transformed by the projection matrix, they are first transformed as you would with any other 4x4 matrix, and the transformed coordinates are then normalized: that is, the x- y- and z-coordinates of the transformed points are divided by the transformed point z-coordinate. Clip space is the space points are in just before they get normalized.

In summary, what happens on a GPU is this.

- Points are transformed from camera space to clip space in the vertex shader. The input vertex is converted from Cartesian coordinates to homogeneous coordinates and its w-coordinate is set to 1\. The predefined <span class="code-inline">gl_Position</span> variable, in which the transformed point is stored, is also a point with homogeneous coordinates. Though when the input vertex is multiplied by the projection matrix, the normalized step is not yet performed. <span class="code-inline">gl_Position</span> is in homogeneous clip space.
- When all the vertices have been processed by the vertex shader, triangles whose vertices are now in clip space are clipped.
- Once clipping is done, all vertices are normalized. The x- y- and z-coordinates of each vertex are divided by their respective w-coordinate. This is where and when the perspective divide occurs.

![](/images/perspective-matrix/vertex-transform-pipeline.png?)

Let's recall, that after the normalization step, points that are visible to the camera are all contained in the range [-1,1] both in x and y. This happens in the last part of the point-matrix multiplication process when the coordinates are normalized as we just said: 

$$\begin{array}{l}
-1 \leq \dfrac{x'}{w'} \leq 1 \\
-1 \leq \dfrac{y'}{w'} \leq 1 \\
-1 \leq \dfrac{z'}{w'} \leq 1 \\
\end{array}
$$

Or: \(0 \leq \dfrac{z'}{w'} \leq 1\) depending on the convention you are using. Therefore we can also write:

$$
\begin{array}{l}
-w' \leq x' \leq w' \\
-w' \leq y' \leq w' \\
-w' \leq z' \leq w' \\
\end{array}
$$

Which is the state x', y' and z' are before they get normalized by w' or to say it differently, when coordinates are in clip space. We can add a fourth equation: \(0 \lt w'\). The purpose of this equation is to guarantee that we will never divide any of the coordinates by 0 (which would be a degenerate case).

These equations mathematically work. You don't need though to try to represent what vertices look like or what it means to work with a four-dimensional space. All it says is that the clip space of a given vertex whose coordinates are {x, y, z} is defined by the extents [-w,w] (the w value indicates what the dimensions of the clip space are). Note that this clip space is the same for each coordinate of the point and the clip space of any given vertex is a cube. Though note also that each point is likely to have its own clip space (each set of x, y, and z-coordinate is likely to have a different w value). In other words, every vertex has its own clip space in which it exists (and needs to "fit" in).

This lesson is only about projection matrices. All we need to know in the context of this lesson is to know where clipping occurs in the vertex transformation pipeline and what clip space means, which we just explained. Everything else will be explained in the lessons on the Sutherland-Hodgman and the Cohen-Sutherland algorithms which you can find in the Advanced Rasterization Techniques section.

## The "Old" Point (or Vertex) Transformation Pipeline

<details>
The fixed-function pipeline is now deprecated in OpenGL and other graphics APIs. Do not use it anymore. Use the "new" programmable GPU rendering pipeline instead. We only kept this section for reference and because you might still come across some articles on the Web referencing methods from the old pipeline.
</details>

Vertex is a better term when it comes to describing how points (vertices) are transformed in OpenGL (or Direct3D, Metal, or any other graphics API you can think of). OpenGL (and other graphics APIs) had (in the old fixed-function pipeline) two possible modes for modifying the state of the camera: GL_PROJECTION and GL_MODELVIEW. GL_PROJECTION allowed setting the projection matrix itself. As we know by now (see the previous chapter) this matrix is built from the left, right, bottom, and top screen coordinates (which are computed from the camera's field of view and near clipping plane), as well as the near and far clipping planes (which are parameters of the camera). These parameters define the shape of the camera's frustum and all the vertices or points from the scene contained within this frustum are visible. In OpenGL, these parameters were passed to the API through a call to `glFrustum` (which we show an implementation of in the previous chapter):

```
glFrustum(float left, float right, float bottom, float top, float near, float far);
```

GL_MODELVIEW mode allowed to set the world-to-camera matrix. A typical OpenGL program set the perspective projection matrix and the model-view matrix using the following sequence of calls:

```
glMatrixMode (GL_PROJECTION);
glLoadIdentity();
glFrustum(l, r, b, t, n, f);
glMatrixMode(GL_MODELVIEW);
glLoadIdentity();
glTranslate(0, 0, 10);
...
```

First, we would make the GL_PROJECTION mode active (line 1). Next, to set up the projection matrix, we would make a call to glFrustum passing as arguments to the function, the left, right, bottom, and top screen coordinates as well as the near and far clipping planes. Once the projection matrix was set, we would switch to the GL_MODELVIEW mode (line 4). Actually, the GL_MODELVIEW matrix could be seen as the combination of the "VIEW" transformation matrix (the world-to-camera matrix) with the "MODEL" matrix which is the transformation applied to the object (the object-to-world matrix). There was no concept of world-to-camera transform separate from the object-to-world transform. The two transforms were combined in the GL_MODELVIEW matrix.

$${GL\_MODELVIEW} = M_{object-to-world} * M_{world-to-camera}$$

First, a point \(P_w\) expressed in **world space** was transformed to **camera space** (or **eye space**) using the GL_MODELVIEW matrix. The resulting point \(P_c\) was then projected onto the image plane using the GL_PROJECTION matrix. We ended up with a point expressed in homogeneous coordinates in which the coordinate w contained the point \(P_c\)'s z coordinate.

## The Vertex Transformation Pipeline in the New Programmable GPU Rendering Pipeline

The pipeline in the new programmable GPU rendering pipeline is more or less the same as the old pipeline, but what is different in this new pipeline, is the way you set things up. In the new pipeline, there is no more concept of GL_MODELVIEW or GL_PROJECTION mode. This step can now be freely programmed in a vertex shader. As mentioned in the first chapter of this lesson, the vertex shader, is like a small program. You can program this vertex shader to tell the GPU how vertices making up the geometry of the scene should be processed. In other words, this is where you should be doing all your vertex transformations: the world-to-camera transformation if necessary but more importantly the projection transformation. A program using an OpenGL API doesn't produce an image if the vertex and its correlated fragment shader are not defined. The simplest form of vertex shader looks like this:

```
in vec3 vert;

void main()
{
    // does not alter the vertices at all
    gl_Position = vec4(vert, 1);
}
```

This program doesn't even transform the input vertex with a perspective projection matrix, which in some cases can produce a visible result depending on the size and the position of the geometry as well as how the viewport is set. But this is not relevant to this lesson. What we can see by looking at this code is that the input vertex is set to be a <span class="code-inline">vec4</span> which is nothing else than a point with homogeneous coordinates. Note that <span class="code-inline">gl_Position</span> too is a point with homogeneous coordinates. As expected, **the vertex shader output the position of the vertex in clip space** (see the diagram of the vertex transformation pipeline above).

In reality, you are more likely to use a vertex shader like this one:

```
uniform mat4 worldToCamMatrix, projMatrix;
in vec3 vert;

void main()
{
    gl_Position = projMatrix * worldToCamMatrix * vec4(vert, 1);
}
```

It uses both a world-to-camera and projection matrix to transform the vertex to camera space and then clip space. Both matrices are set externally in the program using some calls (<span `glGetUniformLocation` to find the location of the variable in the shader and `glUniformMatrix4fv` to set the matrix variable using the previously found location) that are provided to you by the OpenGL API:

```
Matrix44f worldToCamera = ...
// See note below to learn about whether you need to transpose the matrix or not before using it in glUniformMatrix4fv
//worldToCamera.transposeMe();
//projMatrix.transposeMe();
GLuint projMatrixLoc = glGetUniformLocation(p, "projMatrix");
GLuint worldToCamLoc = glGetUniformLocation(p, "worldToCamMatrix");
glUniformMatrix4fv(projMatrixLoc,  1, GL_FALSE, projMatrix);
glUniformMatrix4fv(worldToCamLoc,  1, GL_FALSE, worldToCamera);
```


<details>
_Do I need to transpose the matrix in an OpenGL program or not?_

It is easy to get confused by things such as "should I transpose my matrix before passing it to the graphics pipeline, etc.". In the OpenGL specifications, matrices were/are written using the column-major order convention. Though the confusing part is that API calls such as `glUniformMatrix4vfx()` accept coefficients mapped in memory in the row-major form. In conclusion, if in your code the coefficients of the matrices are laid out in memory in row-major order, then you don't need to transpose the matrix. Otherwise, you may have to. You "may" because this is something you can control via a flag in the `glUniformMatrix4vfx()` function itself. The third parameter of the function which is set to `GL_FALSE` in the example above indicates to the graphics API whether you wish the API to transpose the coefficients of the matrix for you. So even if your coefficients are mapped in memory in column-major order, you don't necessarily need to transpose matrices specifically before using them with `glUniformMatrix4vfx()`. What you can do instead is to set the transpose flag of `glUniformMatrix4vfx()` to `GL_TRUE`.  

Things get even more confusing if you look at the order in which the matrices are used in the OpenGL vertex shader. You will notice we write \(Proj * View * vtx\) instead of \(vtx * View * Proj\). The former form is used when you deal with column-major matrices (because it implies that you multiply the matrix by the point rather than the point by the matrix as explained in our lesson on [Geometry](/lessons/mathematics-physics-for-computer-graphics/geometry/row-major-vs-column-major-vector). Conclusion? OpenGL assumes matrices are column-major (so this is how you need to use them in shaders) yet coefficients are mapped in memory using a row-major order form. Confusing?
</details>

Remember that matrices in OpenGL (and vectors) use column-major order. Thus if you use row vectors as we do on Scratchapixel, you will need to transpose the matrix before setting up the matrix of the vertex shader (line 2). They are other ways of doing this in modern OpenGL but we will skip them in this lesson which is not devoted to that topic. This information can easily be found on the Web anyway.