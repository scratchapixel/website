## A Word of Warning

To understand the content of this lesson, you need to be familiar with the concept of matrix, transforming points from one space to another, perspective projection (including how coordinates of 3D points on the canvas are computed), and with the rasterization algorithm. Read the previous lessons and the lesson on Geometry if you are not familiar with these concepts (see links above).

## Projection Matrices: What Are They?

![Figure 1: multiplying a point by the perspective projection matrices gives another point which is the projection of P onto the canvas.](/images/perspective-matrix/perspprojmatrix1.png?)

What are projection matrices? They are nothing more than 4x4 matrices, which are designed so that when you multiply a 3D point in camera space by one of these matrices, you end up with a new point which is the projected version of the original 3D point onto the canvas. More precisely, multiplying a 3D point by a projection matrix allows you to find the 2D coordinates of this point on the canvas in NDC space. Remember from the previous lesson that the 2D coordinates of a point on the canvas in NDC space are contained in the range [-1, 1]. This is at least the convention that is generally used by graphics APIs such as Direct3D or OpenGL.

<details>
Remember that they are essentially two conventions when it comes to NDC space. Coordinates are either considered to be defined in the range [-1, 1]. This is the case with most real-time graphics APIs such as OpenGL or Direct3D. Or they can also be defined in the range [0, 1]. The RenderMan specifications define them that way. You are entirely free to choose the convention you prefer. We will stick to the convention used by graphics API because this is essential within this context that you will see these matrices being used.
</details>

Another way of saying it is that, multiplying a 3D point in camera-space by a projection matrix, has the same effect as all the series of operations we have been using in the previous lessons to find the 2D coordinates of 3D points in NDC space (this includes the perspective divide step and a few remapping operations to go from screen space to NDC space). In other words, this rather long code snippet which we have been using in the previous lessons:

```
// convert to screen space
Vec2f P_screen; 
P_screen.x = near * P_camera.x / -P_camera.z; 
P_screen.y = near * P_camera.y / -P_camera.z; 
 
// now convert point from screen space to NDC space (in range [-1, 1])
Vec3f P_ndc; 
P_ndc.x = 2 * P_screen.x / (r - l) - (r + l) / (r - l); 
P_ndc.y = 2 * P_screen.y / (t - b) - (t + b) / (t - b); 
```

Can be replaced with a single point-matrix multiplication. Assuming \(M_{proj}\) is a projection matrix, we can write:

```
Vec3f P_ndc; 
M_proj.multVecMatrix(P_camera, P_ndc); 
```

The first version involves five variables: \(near\), \(t\), \(b\), \(l\) and \(r\) which are the near clipping plane, the top, bottom, left, and right screen coordinates respectively. Remember that the screen coordinates are also computed normally from the near-clipping plane as well as the camera angle-of-view (which, if you use a physically-based camera model, is calculated from a whole series of parameters such as the film gate size, the focal length, etc.). This is great because it reduces a rather complex process into a simple point-matrix multiplication operation. The whole point of this lesson is to explain what \(M_{proj}\) is. Though looking at the two code snippet above, should somehow give you some ideas about what we will need to build this matrix. It seems like if this matrix replaces:

- The perspective divide operation,
- As well as remapping the point from screen space to NDC space.

We will somehow have to pack in this matrix all the different variables that are part of these two steps. The near-clipping plane as well as the screen coordinates. We will explain this in detail in the next chapters. Though before we get there, let's explain one important thing about projection matrices and points. First projection matrices are used to transform vertices or 3D points, not vectors. Using a projection matrix to transform a vector doesn't make any sense. These matrices are used to project vertices of 3D objects onto the screen to create images of these objects that follow the rules of perspective. Remember from the lesson on geometry that a point is also a form of matrix. A 3D point can be defined as a [1x3] row vector matrix (1 row, 3 columns). Keep in mind that we use the **row-major order** convention on Scratchapixel. From the same lesson, we know that matrices can only be multiplied by each other if the number of columns of the left matrix equals the number of rows of the right matrix. In other words, the matrices [mxn][nxk] can be multiplied by each other but the matrices [nxm][kxn] can't. Though if you multiply a 3D point with a 4x4 matrix, you get [1x3][4x4], and technically what this means is that this multiplication simply can't be done! The trick to making this operation possible is to treat points not as [1x3] vectors but as [1x4] vectors. Then, you can multiply this [1x4] vector by a 4x4 matrix. Now as usual with matrix multiplication, the result of this operation is another [1x4] matrix. This [1x4] matrix or 4D points in a way are called in mathematics points with **homogeneous coordinates**. A 4D point can't be used as a 3D point unless its fourth coordinate is equal to 1\. When this is the case, the first three coordinates of a 4D point can be used as the coordinates of a standard 3D Cartesian point. We will study this conversion process from Homogeneous to Cartesian in detail in the next chapter. Whenever we multiply a point by a 4x4 matrix, points are always treated as 4D points, but for a reason, we will explain in the next chapters when you use "conventional" 4x4 transformation matrices (the matrices we use the most often in CG to scale, translate or rotate objects for instance), this fourth coordinate doesn't need to be explicitly defined. But when a point is multiplied by a **projection matrix**, such as the perspective or orthographic projection matrices, this fourth coordinate needs to be dealt with explicitly. This is why homogeneous coordinates are more often discussed within the context of projections than within the context of general transformations (even though projections are a form of transformation and even though you are also somehow using homogeneous coordinates when you deal with conventional transformation matrices. You only do so implicitly as we just explained).

<details>
What we call "convention" transformation 4x4 matrices belong to a class of transformation called **affine transformations** in mathematics. Projection matrices belong to a class of transformation called **projective transformations**. To multiply a point by any of these matrices points have to be defined with homogeneous coordinates. You can find more information about homogeneous coordinates, affine and projective transformations in the next chapter and the lesson on [geometry](/lessons/mathematics-physics-for-computer-graphics/geometry/).
</details>

This essentially means that the only time you have to deal with 4D points in a renderer is when you work with projection matrices. The rest of the time, you will never have to deal with them at least explicitly. Projection matrices are also generally only used by programs that implement the rasterization algorithm. In itself, this is not a problem at all, but in the algorithm, there is a process called clipping (we haven't talked about it at all in the lesson on rasterization) that happens **while** the point is being transformed by the projection matrix. You read correctly: clipping, which is a process we will describe in the next chapters, happens somewhere when the points are being transformed by the projection matrix. Not before, nor after. So in essence, the projection matrix is used indirectly to "interleave" a process called clipping that is important in rendering (we will explain what clipping does in the next chapter). And this makes things even more confusing, because generally when books get to the topic of projection matrices, they do also speak about clipping without really explaining why it is there, where it comes from and what relation it has with the projection matrix (in fact it has none, it just happens that it is convenient to do it while the points are being transformed).

## Where Are They Used and Why?

Projection matrices are a very popular topic both on this website but also on specialized forums. They are still very confusing to many and if they are so popular, it must be for something. Not surprisingly, we also found the topic to be generally poorly documented, which is another one of these oddities, considering how important the subject matter is. Their popularity essentially comes from their use in real-time graphics APIs (such as OpenGL, Direct3D, WebGL, Vulkan, Metal, etc.) which are themselves very popular due to their use in games and other common desktop graphics applications. What these APIs have in common is that they are used as an interface between your program and the GPU. Not surprisingly, and as something we already mentioned in the previous lesson, GPUs implement in their circuit the rasterization algorithm. In old versions of the rendering pipeline used by GPUs (known as the fixed function pipeline), GPUs transformed points from camera to NDC space using a projection matrix. But the GPU didn't know how to build this matrix itself. As a programmer, it was your responsibility to build it and pass it on to the graphics card yourself. That essentially meant that you were required to know how to build the matrix in the first place.

```
// Don't use this code - it is now deprecated. OpenGL was
// one the two APIs of choice for real-time graphics (with DirectX).
glMatrixMode(GL_PROJECTION); 
glLoadIdentity(); 
glFrustum(l, r, b, t, n, f);  //set the matrix using screen coordinates and near/far clipping planes 
glMatrixMode(GL_MODELVIEW); 
glLoadIdentity(); 
glTranslate(0, 0, 10); 
...
```

Don't use this code though. It is only mentioned for reference and historical reasons. We are not supposed to use OpenGL that way anymore (these functionalities are now deprecated and OpenGL itself is fading out). In the more recent "programmable" rendering pipeline (DirectX, Vulkan, Metal, or the latest versions of OpenGL), the process is slightly different. First, the projection matrices don't convert points from camera space to NDC space directly, but it converts them into some intermediate space called **clip space**. Don't worry too much about it for now. But to make it short, let's say that in clip space, points have **homogeneous coordinates** (do you remember the 4D points we talked about earlier?). In the modern programmable GPU rendering pipeline, this point-matrix multiplication (the transformation of vertices by the projection matrix) takes place in what we call a **vertex shader**. A vertex shader is nothing else than a small program if you wish, whose job is to transform vertices making up the 3D objects of your scene from camera space to clip space. A simple vertex shader takes a vertex as an input variable, a projection matrix (which is also a member variable of the shader), and set a pre-defined global variable (called `gl_Position` in OpenGL) as the result of the input vertex multiplied by the projection matrix. Note that <span class="code-inline">gl_Position</span> and the input vertex are both declared as `vec4`, in other words as points with homogeneous coordinates. Here is an example of a basic vertex shader:

```
uniform mat4 projMatrix; 
in vec3 position; 
 
void main() 
{ 
    gl_Position = projMatrix * vec4(position, 1.0); 
}
```

![Figure 2: the vertex shader transforms vertices to clip space.](/images/perspective-matrix/rendering-pipeline-vertex.png?)

This vertex shader is executed by the GPU to process every vertex in the scene. In a typical program using the OpenGL or Direct3D API, you store the vertex and the connectivity data (how are these vertices connected to form the mesh's triangles) in the GPU's memory, and the GPU then processes this data (the vertex data in this particular case) to transform them from whatever space they are in when you pass them on to the GPU, to clip space. The space the vertices are in when they are processed by the vertex shader depends on you.

- You can either transform them yourself from world space to camera space before you load them into the GPU's memory. This means that when the vertices will be processed in the vertex shader, the coordinates will already be defined in camera space.
- Or you can leave the vertices in world space before you load them to the GPU's memory. When the vertices will be processed in the vertex shader, beside the projection matrix (which is often denoted P) you will also need to pass on to the shader, the world-to-camera matrix which is often denoted M in the GPU world, where M stands for the "model-view" matrix. This is a matrix that concatenates both the transform from object space to world space and world space to camera space. We didn't speak about object space so far, but this is simply the space an object is in before a transformation is applied to it. World space is the space the object is in after a 4x4 object-to-world matrix has been applied to it. You will first need to transform the vertex from world to camera space and then apply the projection matrix. In pseudo-code you would get something like this:

```
uniform mat4 P;  //projection matrix 
uniform mat4 M;  //model-view matrix (object to wold space * world space to camera space) 
 
in vec3 position; 
 
void main() 
{ 
    gl_Position = P * M * vec4(position, 1.0); 
} 
```

<details>
Remember that OpenGL uses column vector notation (Scratchapixel uses row vector convention). Thus the point that is being transformed appears on the right, and you need to read the transformation from right to left. The vertex is first transformed by M, the model-view matrix, then by P, the projection matrix.
</details>

Many programmers prefer the second option. But what they usually typically do, is to concatenate the world-to-camera and the projection matrix into a single matrix.

```
uniform mat4 PM;  //projection matrix * model-view matrix 
 
in vec3 position; 
 
void main() 
{ 
    gl_Position = PM * vec4(position, 1.0); 
} 
 
int main(...) 
{ 
    ... 
    // we use row-vector matrices
    Matrix44f projectionMatrix(...); 
    Matrix44f modelViewMatrix(...); 
    Matrix44f PM = modelViewMatrix * projectionMatrix; 
    // GL uses column-vector matrices, thus we need to transpose our matrix
    PM.transpose(); 
    // look for a variable called PM in the vertex shader.
    Glint loc = glGetUniformLocation(p, "PM"); 
    // set this shader variable with the content of our PM matrix
    glUniformMatrix4fv(loc,  1, false, &PM.m[0][0]); 
    ... 
    render(); 
    ... 
    return 0; 
} 
```

Don't worry too much if you are not familiar with any graphics API. Or maybe you are familiar with Direct3D but not with OpenGL, though the principles are the same, so you should be able to find similarities easily. Regardless, just try to understand what the code is doing. In the main CPU program, we just set the projection matrix and the model view matrix (which combines the object-to-world and the work-to-camera transform), and multiply these two matrices together so that rather than passing two matrices to the vertex shader, only one is needed. The OpenGL API required that we find the location of the variable we are looking for in a shader using the `glGetUniformLocation()` call.

<details>
Don't worry if you don't know how `glGetUniformLocation()` works. But in short, it takes a program as the first argument and the name of a variable you are looking for on this program and returns the location of this variable. You can then later use this location to set the shader variable using a <span class="code-inline">glUniform()</span> call. A program in OpenGL or Direct3D combines a vertex shader and a fragment shader. You first need to combine these shaders in a program, and the program is then applied or assigned to an object. Both shaders define how the object is transformed from whatever space the model is in when the vertex shader is executed to clip space (this is the role of the vertex shader) and then define its appearance (that's the function of the fragment shader).
</details>

Once the location is found, we can set this shader variable using the `glUniformMatrix4fv()`. When the geometry will be rendered, the vertex shader will be executed which as a result, will transform vertices from object space to clip space. It will do so by multiplying the vertex position (in object space) by the matrix PM which combines the effect of the projection matrix with the model-view matrix. Hope this all makes sense. **This process is central to the way objects are rendered on GPUs**. Though this lesson is not an introduction to the GPU rendering pipeline, thus we don't want to get into too much detail at this point, but hopefully, these explanations are enough for you to at least understand the concept. And regardless of whether you use the old or the new rendering pipeline, you are still somehow required to build the projection matrix yourself. And this is why projection matrices in general are so important and consequently why they are so popular on CG forums. Now, note that in the new rendering pipeline, using a projection matrix is not required. All that is required is that you transform vertices from whatever space they are in, to clip space. Whether you use a matrix for this, or just write some code that does the same thing as what the matrix does, is not important as long as the result is the same. For example, you could very much write:

```
uniform float near, b, t, l, r;  //near clip plane and screen coordinates 
 
in vec3 position; 
 
void main() 
{ 
    // does the same thing than a gl_Position.x = Mproj * position
    gl_Position.x = ... some code here ...; 
    gl_Position.y = ... some code here ...; 
    gl_Position.z = ... some code here ...; 
    gl_Position.w = ... some code here ...; 
}
```

Hopefully, you understand what we mean when we say you can either use the matrix or just write some code that sets <span class="code-inline">gl_Position</span> as if you had used a projection matrix. This wasn't possible in the fixed-function pipeline (because the concept of vertex shader didn't exist back then) but the point we are trying to make here, is that it is not strictly required anymore to use projection matrices if you do not wish to. The advantage of being able to replace it with some other code is that it gives you more flexibility on the way vertices are transformed. Though in 99% of the cases you will be more likely to use a standard projection than not and even if you do not wish to use the matrix, you will still need to understand how the matrix is built to replace the point-matrix multiplication with some equivalent code (and transform the vertex coordinates one by one).

![Figure 3: passing on the projection matrix and the vertex data from the CPU to the GPU pipeline.](/images/perspective-matrix/rendering-pipeline-vertex2.png?)

All you need to remember really from this chapter is that:

- GPUs' rendering pipeline is based on the rasterization algorithm, in which vertices are typically transformed from camera space to clip space using projection matrices. GPUs are widespread and since projection matrices play a central role in the way images are produced on the GPU, they have also become an important topic of discussion and interest. Another way of saying this is that if GPUs didn't exist or if they were based on the ray-tracing algorithm instead, we would probably don't care about projection matrices at all.
- In the programmable rendering pipeline, vertices are transformed to clip space in the vertex shader. The projection matrix is generally a member variable of the vertex shader. The matrix is set on the CPU before the shader is used (you can also pass to the shader all the variables that it needs to build the matrix itself, though this approach is not the most efficient). All we need to know is to learn how this matrix is built. And what clip space is.

## Orthographic and Perspective Projection Matrix

![](/images/perspective-matrix/sim-city.jpg?)

Finally and to conclude this chapter, you may have noticed that the lesson is called "The Perspective and Orthographic Projection Matrix", and you may wonder what the difference is between the two. First, let's say that they are both projection matrices. In other words, their function is to somehow project 3D points onto a 2D surface. We are already familiar with the perspective projection process. When the angle of view becomes very small (in theory when it tends to 0) then the four lines defining the frustum's pyramid get parallel to each other. In this particular configuration, there is no more foreshortening effect. In other words, an object's size in the image stays constant regardless of its distance from camera. This is what we call an **orthographic projection**. Visually, we find an orthographic projection less natural than a perspective projection, however, this form of projection is nonetheless useful in some cases. Sim City is also an example of a game that was rendered with an orthographic projection (image above). This gives the game a unique look.

![](/images/perspective-matrix/projectionsexample.png?)

## Projection Matrix and Ray-Tracing

As we mentioned a few times already, in ray tracing, points don't need to be projected onto the screen. Instead, rays emitted from the image plane are tested for intersections with the objects from the scene. Thus, projection matrices are not used in ray-tracing.

## What's Next?

We found that the easiest way to learn about projection matrices and all the things we just talked about, is to start by learning how to build a very basic one. Once you understand what these homogeneous coordinates are, it will also become simpler to speak about clipping and other concepts which indirectly relate to projection matrices. The simple perspective projection matrix that we will build in chapter three, won't be as sophisticated as the perspective projection matrix used in OpenGL or Direct3D (which we will also study in this lesson). As mentioned, the goal of chapter three is just to explain the principle behind projection matrices. In other words, how they work. We said that a projection matrix remaps vertices from whatever space they were in to clip space. Though, for the next two chapters, we will get back to our basic definition of the projection matrix which is that it converts points from camera space to NDC space. We will study the more generic case or more complex case in which points are converted to clip space in chapter four (in which we will explain how the OpenGL matrix works). For now, don't worry about clip space and keep this definition in mind: "projection matrices convert vertices from camera space to NDC space" (though keep in mind that this definition is only temporary).