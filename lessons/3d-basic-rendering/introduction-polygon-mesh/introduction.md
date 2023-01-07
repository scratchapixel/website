![Figure 1: polygon components.](/images/polygon-mesh/mesh-components.png?)

![Figure 2: convex vs. concave.](/images/polygon-mesh/mesh-concave-convex.png?)

![Figure 3: polygon with a hole.](/images/polygon-mesh/mesh-poly-hole.png?)

![Figure 4: a cube can only be defined with 8 vertices.](/images/polygon-mesh/mesh-cube.png?)

![Figure 5: we only need 4 vertices to define 2 connected triangles.](/images/polygon-mesh/mesh-tris2.png?)

In the first few lessons devoted to rendering, we rendered a polygon mesh to demonstrate a few techniques such as the rasterization or ray-tracing algorithm, but we didn't provide any information, on what is a polygonal mesh, and how is the information about this mesh stored either on disk or in the memory of a program. This lesson intends to fill this gap.

Polygon meshes or meshes for short are probably the oldest forms of geometry representation used in computer graphics. The idea is simple. It is based on the elementary brick called a polygon or face. A polygon is just a "planar" shape, which is defined by connecting a series of 3D vertices (Figure 1).

The individual dots or points are called vertices (the singular is vertex). In 2D they are defined by their x and y-coordinates and in 3D by their x, y, and z-coordinates. The face is defined by connecting these vertices (either in counter-clockwise or clockwise order - the ordering is very important and plays a role in computing for instance the normal direction. It is also called winding - more information on winding can be found below). The line connecting two vertices is called an edge. A face can have a minimum of 3 vertices. This is then a **triangle**. If you have four vertices it is called a **quad**, and if you have more than four vertices, then it is simply a **general polygon**. Note that generally, you want the vertices making up a face to be in the same plane. This is always the case for triangles (three vertices define a plane), but this is not necessarily the case when you have more than three vertices. A polygon can be either convex or concave as shown in the image below.

To make things more interesting (more complex), polygons may also have holes. But we won't be considering this case in this lesson.

Now that we have defined what a face or polygon is, we can create more complex 3D shapes by simply connecting these faces. A cube for example can be created by arranging six faces together. Note that, all you need to describe a cube, is just to define the position of its 8 vertices, and then to define how are these vertices connected to form the cube's face. The way vertices are connected is simply called **connectivity**. Vertex and connectivity information is the minimum amount of data required to define 3D shapes in CG.

From a programming point of view, there are different ways of representing this connectivity information. Most techniques attempt to minimize the amount of data that is needed to define how vertices are connected, in order to reduce the size of the files on disk, and the amount of memory used when the files are loaded in memory (you can also store the data in a certain way for performance reasons). We won't detail these advanced techniques in this lesson. Though what we will talk about is the standard way of storing this information which is both simple and generic. Most standard file formats (FBX, Obj) or API specifications (such as the RenderMan specifications) use this technique in one way or another. Let's detail it now.

Note that even though a cube has 6 faces, we don't need to explicitly declare 24 vertices (declare 6 times the 4 vertices making up each face). What we can do instead is to declare the 8 vertices of the cube (8 vertices instead of 24 is a big saving already), and for each face of the cube, we will specify which ones of these vertices are used (the connectivity information we talked about earlier). Let's see a simple example. Let's imagine that two triangles share an edge as shown in figure 4. Because the triangles share 2 vertices, rather than declaring 2 times 3 vertices, we just need to declare 4 vertices.

```
Vec3f verts[4] = {{-1,0,0}, {0,1,0}, {1,0,0}, {0,-1,0}};
```

To compose the first triangle, we will need to index in the array above the following vertices: 0, 2, 1. And for the second triangle, we will need to index the vertices with indices: 0, 3, and 2 (note that we always connect vertices in counterclockwise order). The only problem with this approach is that while we reduce the number of vertices being declared, we need an additional array of indices to store connectivity data.

```
int indices[6] = {0, 2, 1, 0, 3, 2};
```

Let's review what we need:

- We first need to know how many polygons the object is made of in the first place. This would be 6 for a cube. In our simple example (made of two triangles) this number would be 2.
- We then need to know for each face of the object, how many vertices each consecutive face is made of. For the cube, this would be an array of 6 integers having the following values {4, 4, 4, 4, 4, 4}. In our triangle example, this would be {3, 3}. Note that the size of this array is equal to the number of polygons the mesh contains. Let's call this array the face index array.
- We also need to know for each face, the index of each vertex in the vertex array making up the face. For the triangle example this array would look as follows: {0, 2, 1, 0, 3, 2}. We know the first face has 3 vertices. Thus we just read the first 3 integer values from this array (the values 0, 2, 1) which are then used to index the following vertices in the vertex array: `{-1,0,0}, {0,1,0}, {1,0,0}, {0,-1,0}}`. The next face is also a triangle, thus we read the next 3 values from the vertex index array (the values 0, 3, 2) and read the vertex position from the vertex array using these indices: `{-1,0,0}, {0,1,0}, {1,0,0}, {0,-1,0}}`. Let's refer to this array as the vertex index array. Note that the size of this array is equal to the sum of all the integer values from the face index array (3 + 3 in the triangle example, and 6 times 4 for the cube).
- Finally, we need to know the position of the vertices (we called the vertex array above). Note that each face of the cube shares some vertices with other faces (as well as edges). Let's refer to this array as the vertex array.

|-table{Name,Description,Example}
|-row
|-cell
Face Index Array
|-cell
Number of vertices each face is made of
|-cell
`{3, 3}`
|-row
|-cell
Vertex Index Array
|-cell
Indices of the vertices making up each face in the vertex array</td>
|-cell
`{0, 2, 1, 0, 3, 2}`
|-row
|-cell
Vertex Array
|-cell
Vertex position
|-cell
`{{-1,0,0}, {0,1,0}, {1,0,0}, {0,-1,0}}`
|-

Let's see what this would look like for a cube:

To learn one possible but easy way of defining this information, we will use the RenderMan Interface for inspiration (as a comparison point, we will look into how this information is stored in OBJ and FBX files). See the information note below if you don't know what OBJ, FBX, or the RenderMan Interface are.

<details>
What is the [RenderMan Interface](http://renderman.pixar.com/products/rispec/rispec_pdf/RISpec3_2.pdf). It is a document that was first published by Pixar in 1989. The goal of the document was to propose a standard for passing on scene data to any renderer (the interface between your 3D programs such as Maya and your renderer such as RenderMan). Scene data would include geometry data, but also camera data or global data such the image size for example. The document was last updated in 2005 (version 3.2.1 of the interface). Pixar is not maintaining it anymore for various reasons, though the document itself stays a very good starting point for anyone interested in learning how scene data can be passed to a renderer). 

[OBJ](http://en.wikipedia.org/wiki/Wavefront_.obj_file) is a "simple" object or geometry file format originally developed by Wavefront. It can only store information about polygon meshes, and another type of surfaces (it is a geometry file format while the RenderMan specification defines to describes the content of an entire image or sequence of images including the geometry, camera, lights, etc.).

[FBX](http://en.wikipedia.org/wiki/FBX) is a more complex and generic file format owned and developed by Autodesk which is used to exchange scene data between applications. It supports things such as lights, materials, cameras, animation as well as geometry data. The advantage of the RenderMan interface is that it is well-suited for rendering. The advantage of the OBJ format is its simplicity. The advantage of FBX is that it is a standard in the industry at the moment.

Many other 3D formats exist such as glTF, Alembic (more like a format to store cached animation data), USD (also a format initially developed by Pixar), etc.
</details>

```[wrap]
PointsPolygons [4 4 4 4 4 4] [0 1 3 2 2 3 5 4 4 5 7 6 6 7 1 0 1 7 5 3 6 0 2 4] "P" [-0.5 -0.5 0.5 0.5 -0.5 0.5 -0.5 0.5 0.5 0.5 0.5 0.5 -0.5 0.5 -0.5 0.5 0.5 -0.5 -0.5 -0.5 -0.5 0.5 -0.5 -0.5]
```

So how does the RenderMan Interface defines a polygon mesh? First the keyword `PointsPolygons` is used to tell the program that whatever information comes next defines a polygon mesh. The first array of integers is our face index array. It defines the number of vertices each face of the object is made of. The size of the array (6 in our example) is also the number of polygons making up the object as we know by now (we don't need to declare the number of polygons explicitly, we can use the size of this array instead). By looking at this array, we can tell that our object has 6 faces (the size of the array) and that each face making up this object has 4 vertices. The second array is the vertex index array. The first four integers are the indices of the vertices in the vertex array making up the first face of the object, and so on. Finally, the vertex's positions are defined in the array following the symbol "P". "P" in the interface stands for point. The RenderMan interface uses this convention to define as many "primitive variables" as you'd like. "P" is the only primitive variable required. All the others such as normal information (generally denoted "N") or texture coordinates (generally denoted "st") are optional. We will see an example with additional primitive variables later.

Note that all arrays are 0-based. While the vertex array in this example has exactly 8 vertices nothing stops you from having more than 8 vertices if you would like to. For example, a triangle only needs 3 vertices, but you could very much have:

```
int faceIndex[1] = {3}; int vertexIndex[5] = {0, 5, 3}; Vec3f verts[6] = {v0, v1, v2, v3, v4, v5};
```

In other words, we could declare an array of 6 vertices, and have the indices of the vertex index array pointing to any one of these 6 vertices in the vertex array. This would be a perfectly valid declaration. The only obligation here is that the maximum index value (plus one since we use a 0-based array) in the vertex index array is no greater than the size of the vertex array (in our example 5 + 1 = 6, which is good). This is a detail, but worth noting.

As just mentioned above, you may want to declare more than just the vertex position for your model. For example, you may also want to declare things such as vertex color, texture coordinates, or normals. These are called primitive variables in the RenderMan terminology though this is generally how they are called in the industry now (a [primitive](http://en.wikipedia.org/wiki/Geometric_primitive) in computer graphics is also the generic term given to geometric objects that a given program can handle). In the RenderMan interface a per-vertex or per-face primitive variable is declared using the keyword `facevarying`. If the variable is per-face though this keyword is preceded with the keyword `uniform`. You of course also need to declare the type of the variable. "P" is predefined and its type is implicit (point). But you need to declare the type of all the other variables. For example in the example below "Ns" is declared as normal and the st coordinates (which are declared as two separate arrays of floats) have the type float. Here is an example:

```[wrap]
PointsPolygons [4 4 4 4 4 4] [0 1 3 2 2 3 5 4 4 5 7 6 6 7 1 0 1 7 5 3 6 0 2 4] "P" [-0.5 -0.5 0.5 0.5 -0.5 0.5 -0.5 0.5 0.5 0.5 0.5 0.5 -0.5 0.5 -0.5 0.5 0.5 -0.5 -0.5 -0.5 -0.5 0.5 -0.5 -0.5] "facevarying normal N" [0 0 1 0 0 1 0 0 1 0 0 1 0 1 0 0 1 0 0 1 0 0 1 0 0 0 -1 0 0 -1 0 0 -1 0 0 -1 0 -1 0 0 -1 0 0 -1 0 0 -1 0 1 0 0 1 0 0 1 0 0 1 0 0 -1 0 0 -1 0 0 -1 0 0 -1 0 0] "facevarying float s" [0.375 0.625 0.625 0.375 0.375 0.625 0.625 0.375 0.375 0.625 0.625 0.375 0.375 0.625 0.625 0.375 0.625 0.875 0.875 0.625 0.125 0.375 0.375 0.125] "facevarying float t" [0 0 0.25 0.25 0.25 0.25 0.5 0.5 0.5 0.5 0.75 0.75 0.75 0.75 1 1 0 0 0.25 0.25 0 0 0.25 0.25] "constant string primtype" ["mesh"]
```

This is not a lesson on the RenderMan Interface, so we won't detail this any further. Though hopefully, this is enough for you to get a sense of the sort of controls you need to carry information about a polygon mesh (we highly recommend you to have a look at the RenderMan Interface document though).

## How does this work in code?

That is a straight application of the concepts we studied earlier. Here is for example how you would access the vertices of the cube:

```
const int numVerts = 8; 
const int numFaces = 6; 
int faceIndex[numFaces] = {4, 4, 4, 4, 4, 4}; 
int vertexIndex[24] = {0, 1, 2, 3, 0, 4, 5, 1, 1, 5, 6, 2, 0, 3, 7, 4, 5, 4, 7, 6, 2, 6, 7, 3}; 
Vec3f verts[numVerts] = {{-1,1,1},{1,1,1},{1,1,-1},{-1,1,-1},{-1,-1,1},{1,-1,1},{1,-1,-1},{-1,-1,-1}}; 
int offset = 0; 
for (int i = 0; i < numFaces; ++i) { 
    std::cerr << "Face: " << i << " has " << faceIndex[i] << " vertices\n"; 
    for (int j = 0; j < faceIndex[i]; ++j) { 
         int vertIndex = vertexIndex[offset + j]; 
         std::cerr << j << " vertex index: " << vertIndex << " pos: " << verts[verIndex] << "\n"; 
    } 
    offset += faceIndex[i]; 
}
```

## The Most Common Primitive Variables

Before we look into some other file formats such as FBX and OBJ, we will first have a look at what normals and texture coordinates are (how they are used). Note that you can as many primitive variables as you want, though these two are the most two common optional ones (besides vertex positions which as we know is mandatory). Vertex color was used a lot in the past (essentially for games), but since the advent of the GPU programmable architecture, it is not as commonly used anymore.

### Normals

We already introduced the concept of normal in the lesson on geometry. It is a special kind of vector that defines the orientation of a surface. The vector is generally perpendicular to the surface at the point where the normal is defined. Though the normal doesn't have to be perpendicular. This is sometimes used to perform some shading tricks. Though, in general, a normal, by definition, is perpendicular to the surface at the point where it is defined. Normals are a special kind of vector because they are not transformed by 4x4 affine transformation matrices the same way as vectors are. Please check the [lesson on geometry for further detail](/lessons/mathematics-physics-for-computer-graphics/geometry/transforming-normals). They need to be transformed instead by the inverse of the matrix. You don't have to create a special "normal" type to store normal information. You can very much use something similar to `Vec3f`. The only advantage of using a normal type is to make it easier to handle situations in which they are transformed by matrices. For example, you could do something like this:

```
struct Transform 
{ 
    Matrix44f M; 
    Matrix44f Minv; 
}; 
 
typename Vec3<float> normal; 
typename Vec3<float> vector; 
typename Vec3<float> point; 
 
void transformPoint(const point &in, point &out, const Transform &T) 
{ 
    out.x   = in.x * T.M[0][0] + in.y * T.M[1][0] + in.x * T.M[2][0] + T.M[3][0]; 
    out.y   = in.x * T.M[0][1] + in.y * T.M[1][1] + in.x * T.M[2][1] + T.M[3][1]; 
    out.z   = in.x * T.M[0][2] + in.y * T.M[1][2] + in.x * T.M[2][2] + T.M[3][2]; 
    float w = in.x * T.M[0][3] + in.y * T.M[1][3] + in.x * T.M[2][3] + T.M[3][3]; 
    if (w != 1) { 
         out.x /= w; 
         out.y /= w; 
         out.z /= w; 
    } 
} 
 
void transformVector(const vector &in, vector&out, const Transform &T) 
{ 
    out.x = in.x * T.M[0][0] + in.y * T.M[1][0] + in.x * T.M[2][0] + T.M[3][0]; 
    out.y = in.x * T.M[0][1] + in.y * T.M[1][1] + in.x * T.M[2][1] + T.M[3][1]; 
    out.z = in.x * T.M[0][2] + in.y * T.M[1][2] + in.x * T.M[2][2] + T.M[3][2]; 
} 
 
void transformPoint(const normal &in, normal &out, const Transform &T) 
{ 
    out.x = in.x * T.Minv[0][0] + in.y * T.Minv[1][0] + in.x * T.Minv[2][0] + T.Minv[3][0]; 
    out.y = in.x * T.Minv[0][1] + in.y * T.Minv[1][1] + in.x * T.Minv[2][1] + T.Minv[3][1]; 
    out.z = in.x * T.Minv[0][2] + in.y * T.Minv[1][2] + in.x * T.Minv[2][2] + T.Minv[3][2]; 
} 
```

This is perfectly valid and probably the "right" way of doing it, though lots of programmers just prefer not to bother with the complication.

What are normals used for? Essential for shading. The orientation of the surface plays a major role in the amount of light that is reflected toward the eye. We generally consider the angle between the normal and the incoming light direction and the angle between the normal and the eye (also called viewing direction). These two angles define how much of the incoming light is reflected toward the viewer. Check the lesson called [Introduction to Shading](/lessons/3d-basic-rendering/introduction-to-shading/) in the basic rendering section to learn more about the use of normals in shading.

Normals are either defined per face or per vertex. A simple way of computing a per-face normal is to simply take any two edges making the face and use these two vectors in a cross-product. Though not that this only gives consistent results if the face is coplanar (which you can guarantee by triangulating your mesh). Note also that the direction of the normal depends on the winding and handedness convention you are using. Let's consider the case of this basic triangle with vertices (it lies in the x-y plane):

```
Vec3f verts[3] = {[-1,0,0}, {0,1,0}, {1,0,0}};
```

If we assume a counter-clockwise ordering, then the index making up this triangle would be 0, 2, 1 (figure 4). Let's compute the cross product between the edges \(E_{02}\) and \(E_{21}\).

```
E02 = { 2, 0, 0};
E21 = {-1, 1, 0};

x = E02.y * E21.z - E02.z * E21.y = 0 *  0 - 0 *  1 = 0;
y = E02.z * E21.x - E02.x * E21.z = 0 * -1 - 2 *  0 = 0;
z = E02.x * E21.y - E02.y * E21.x = 2 *  1 - 0 * -1 = 2;
```

The resulting normal is {0, 0, 2} which after normalization gives {0, 0, 1}.

But if we were to declare the vertices in clockwise order, we would get (we would need to compute the cross product between E01 and E12):

```
E01 = {1,  1, 0};
E12 = {1, -1, 0};

x = E01.y * E12.z - E01.z * E12.y = 1 *  0 - 0 * 1 =  0;
y = E01.z * E12.x - E01.x * E12.z = 0 *  1 - 1 * 0 =  0;
z = E01.x * E12.y - E01.y * E12.x = 1 * -1 - 1 * 1 = -2;
```

The resulting normal is {0, 0, -2} which after normalization gives {0, 0, -1}.

![Figure 6: face and vertex normals.](/images/polygon-mesh/mesh-normal-face.png?)

In conclusion, the normals point in opposite direction. This is why winding and handedness matter when you compute normals from the face's edges. Remember that on Scratchapixel, we use a right-hand coordinate system (RenderMan by default assume a left-hand coordinate system and clockwise ordering). We have learned to compute face normals. Though normal information can also be declared at the vertex position. For example, a cube has 6 faces. Thus it would have 6 face normals though a vertex can have as many normals as faces it is connected to. Each vertex of a cube is connected to 3 faces and thus can have 3 vertex normals, one for each face. In total, a cube has 24 vertex normals (Figure 6).

Remember that vertex primitive variables are interpolated across the surface of the face when the object is rendered. This method was explained in the lesson on [Rasterization](/lessons/3d-basic-rendering/rasterization-practical-implementation/rasterization-stage) and [Ray-Tracing](/lessons/3d-basic-rendering/ray-tracing-rendering-a-triangle/barycentric-coordinates). Vertex normals too are interpolated across the surface of the face. Check the introduction on shading in the basic rendering section to learn more about vertex normal and their use in shading (and look up smooth shading).

### Texture Coordinates

![Figure 7: texture mapping is a technique in which an image is applied to the surface of a 3D object to add surface detail.](/images/polygon-mesh/texture-mapping.gif?)

Texture mapping is a technique used in computer graphics to add details to the surface of objects. Generally, we do so by mapping an image on its surface though we can also use a technique called procedural texturing in which the pattern applied to the surface of the object is procedurally generated. The process is similar in a way to applying wallpaper on a wall. 3D objects though are not as simple as the surface of a wall. However, we can see each face making up a mesh as a co-planar surface on which we can apply a section of that wallpaper. If you imagine that the bottom-left corner of the wallpaper strip defines the origin of a 2D cartesian coordinate system (see image below), we can then extract a face from the polygon mesh and apply it on the surface of that wallpaper. The position of the face on the wallpaper would determine which part of that wallpaper would be applied to the face (as shown in the illustration below). Note also that each vertex making up that face has coordinates defined with respect to the 2D cartesian coordinate systems we mentioned before. These 2D coordinates are what we call in CG, **uv**, **st**, or **texture coordinates**. The process of applying the wallpaper which is CG is going to be a digital image is called **texture mapping** or **texturing** for short. The process is illustrated in the figure below

![](/images/polygon-mesh/mesh-uv-coordinates2.png?)

![Figure 8: 2D texture coordinates defined with respect to a Cartesian coordinate system whose origin is located in the lower-left corner of the square image.](/images/polygon-mesh/mesh-uv-coordinates4.png?)

The face of the mesh is applied on the surface of a square image (our digital wallpaper). We apply the face on the surface of our digital wallpaper, and use its shape to cut out a piece of the wallpaper which we then apply to the surface of the face on the 3D model. This is the principle of texture mapping. The uv-coordinates of the face, are 2D coordinates that define the position of the face on the surface of the image with respect to the uv-space coordinate system whose origin is in the bottom-left coordinate of the image.

Now that we understand the principle for one face of the mesh, we can repeat the same process for every face of the mesh. In the end, each face is applied to a piece of the digital image. The process by which faces are laid out on the surface of the digital wallpaper which we will call a **texture** from now on is called **uv layout**. As an example, the uv coordinates of the face which we mapped in the example above, are indicated in Figure 8. In programming, the uv-coordinates of the face used in our example would be declared as follows:

```
Vec2f tex[5] = {{0.19, 0.55}, {0.32, 0.55}, {0.35, 0.44}, {0.25, 0.36}, {0.16, 0.44}};
```

In most cases, uv-coordinates are defined in the range [0, 1] which means that the texture is a square (but textures coordinates don't have to be limited to the range [0, 1]. Check the lesson on texture mapping for more information).

There are several observations we can make regarding texture coordinates:

- In the example above, the shape of the mesh on the surface of the texture is the same as the shape of the face in 3D space. Though it is entirely possible to move the vertices of the face in 2D space to create a different shape. If the shape of the face in uv space is different than the shape of the face in 3D space, then the texture on the surface of the face in 3D space is likely to be stretched.
- In the same vein that the previous note, you can apply a translation, a rotation, and a scale to uv coordinates. By translating the uv-coordinates the texture slides across the surface of the face. Rotation rotates the texture on the face. By scaling the coordinates, you can either zoom in or zoom out on the texture.
- Several faces can be mapped to the same area of the texture. To say it differently, texture coordinates can overlap.
- Texture coordinates are not limited to the range [0, 1], though this is the most common case.

For more information on this topic, please refer to the lesson on texture mapping in the basic rendering section. For now, let's see how a polygon mesh defines texture coordinates.

From a programming point of view, uv's are exactly like faces. We first need to declare the coordinates themselves as an array of 2D vertices, and then define an array similar to the vertex index array but specific to the texture coordinates. Let's call this array the texture index array. In the following table, we reused the example of our two connected triangles:

|-table{Nane,Description,Example}
|-row
|-cell
Face Index Array
|-cell
Several vertices each face is made of.
|-cell
`{3, 3}`
|-row
|-cell
Texture Index Array (optional)
|-cell
Indices of the vertices make up each face in the texture coordinates array.
|-cell
`{0, 2, 1, 0, 3, 2}`
|-row
|-cell
Texture Coordinates Array
|-cell
Vertex position in uv-space.
|-cell
{{0,0.5}, {0.5,1}, {1,0.5}, {0.5,0}} or 6 coordinates if you don't use a texture index array  
{{0,0.5}, {1,0.5}, {0.5,1}, {0,0.5}, {0.5,0}, {1,0.5}}.
|-

![Figure 9: layout of the cube uv's.](/images/polygon-mesh/mesh-cube-uvs.png?)

Note that the face index array is the same for both the faces in 3D space and in uv-space (thus it only needs to be declared once). The faces have the same number of vertices in both 2D and 3D spaces. That is logical. The order in which the vertices are connected for each face doesn't have to be the same in 3D and uv-space. Though in the RenderMan specification, it is assumed that the number of texture coordinates is equal to the sum of all the face's vertices and that they are declared in the order in which the faces are constructed. For example, if you have 3 faces with 4, 3, and 5 vertices respectively, you need to declare a total of 12 texture coordinates. The first 4 coordinates will be the coordinates of the first face, the next 3 coordinates will be the uv coordinates of the second face, and so on. The texture coordinate array, of course, is just an array of 2D points.

Here is an example of a cube defined in the RenderMan interface. Note that the uv-coordinates in RenderMan are called st and are declared as two 1D arrays, as opposed to a single 2D array. There is no particular reason for this. It is just a convention:

```[wrap]
PointsPolygons [4 4 4 4 4 4] [0 1 3 2 2 3 5 4 4 5 7 6 6 7 1 0 1 7 5 3 6 0 2 4] "P" [-0.5 -0.5 0.5 0.5 -0.5 0.5 -0.5 0.5 0.5 0.5 0.5 0.5 -0.5 0.5 -0.5 0.5 0.5 -0.5 -0.5 -0.5 -0.5 0.5 -0.5 -0.5] "facevarying normal N" [0 0 1 0 0 1 0 0 1 0 0 1 0 1 0 0 1 0 0 1 0 0 1 0 0 0 -1 0 0 -1 0 0 -1 0 0 -1 0 -1 0 0 -1 0 0 -1 0 0 -1 0 1 0 0 1 0 0 1 0 0 1 0 0 -1 0 0 -1 0 0 -1 0 0 -1 0 0] "facevarying float s" [0.375 0.625 0.625 0.375 0.375 0.625 0.625 0.375 0.375 0.625 0.625 0.375 0.375 0.625 0.625 0.375 0.625 0.875 0.875 0.625 0.125 0.375 0.375 0.125] "facevarying float t" [0 0 0.25 0.25 0.25 0.25 0.5 0.5 0.5 0.5 0.75 0.75 0.75 0.75 1 1 0 0 0.25 0.25 0 0 0.25 0.25] "constant string primtype" ["mesh"]
```

You can see what the layout of the faces of this cube in uv space looks like in figure 8.

## Summary

Let's summarize the information that you will typically need to describe a polygon mesh. The required data to define an object, are the vertex positions, the face index array (how many vertices each face is made of), and the vertex index array. You can also add primitive variables such as normals and uv coordinates. Primitive variables can either be defined per face or per vertex.

The way you declare and access this information depends on the file format you use to store and retrieve the object data. For example, RenderMan splits the texture coordinates into two 1D arrays. But it uses only one vertex index array for vertex positions, normals, and texture coordinates (and any other declared primitive variables). Though, some other formats (such as OBJ) can use different indices for the vertices, normals, and texture coordinates.

## What's Next?

In the next chapter, we will study how the most common 3D file formats (OBJ, FBX, RenderMan, etc.) store 3D mesh data.