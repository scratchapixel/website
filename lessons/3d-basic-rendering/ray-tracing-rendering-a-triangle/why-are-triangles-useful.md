In theory, solving ray and triangle intersections is not all that difficult. What makes the ray-triangle intersection complex is the multitude of different cases that we must account for. Writing a robust routine that handles all these cases quickly and efficiently can be tricky. For this reason, the topic has been the subject of much research and debate in the CG community. It is also generally one of the most critical routines in a ray-tracer. In this lesson, we will stick to the basic techniques and foundations upon which more advanced methods are built.

## Geometric Primitives

A **geometric primitive** represents a 3D object in a rendering program. For example, suppose we want to represent a sphere. In that case, we can define it by the position of its center and its radius as explained in the [previous lesson](/lessons/3d-basic-rendering/minimal-ray-tracer-rendering-simple-shapes/ray-sphere-intersection). Complex objects, similarly, can be modeled by more complex geometric primitives like **polygons**, **NURBS or Bezier patches**, **subdivision surfaces**, etc. Each one of them is useful for representing certain types of 3D objects. For example, NURBS are suitable for objects with smooth surfaces. Polygons, on the other hand, are helpful for geometric shapes like buildings. A triangle is not a geometry type of its own. It is rather a subset of the polygon primitive type. We will see in a later lesson that a polygonal object is easily convertible into triangles.

## Why Do We Like Triangles so Much?

![Figure 1: geometry of a triangle. A triangle is defined by three vertices that define a plane. The normal is perpendicular to this plane. A ray (in yellow) intersects this triangle.](/images/ray-triangle/triangle1.png)

Calculating a ray's intersection with a primitive such as a sphere is not tricky. However, since it is challenging to model most 3D objects with spheres alone, it is necessary to use some other types of primitive to represent more complex objects (objects of arbitrary shape). Nothing stops us from using polygonal meshes or NURBS surfaces with our renderer, but computing the intersection of a ray with these primitives can be difficult and slow. On the other hand, computing the intersection of a ray with a triangle is a relatively simple operation that can also be well-optimized. It may not be as fast as computing an intersection between a ray and a sphere, but at least it will be better than rendering the intersection of a ray with a NURBS surface. This is a compromise we are willing to make.

Instead of working with complex primitives such as NURBS or Bezier patches, we can convert every object into a triangle mesh and compute the intersection of a ray with every triangle in this mesh. Doing so reduces the ray-object intersection to a single, advantageously fast routine. Converting a Bezier patch into a triangle mesh is much more straightforward than computing multiple ray-Bezier patch intersections. In our code, the consequence of this strategy is that every time we implement a new geometric primitive type, we will have to write a routine that will convert that type into a triangle mesh instead of a routine to compute the intersection of a ray with this primitive. Therefore, all the objects of a scene will be internally represented as triangle meshes. Most ray tracers (at least production ones) use this approach.

## What Is a Triangle?

![Figure 2: a quad is not necessarily coplanar. The four vertices it is made of are not lying in the same plane (as depicted on the right). To solve this problem, we can triangulate quads as we know that triangles are necessarily coplanar (on the left, the right quad is converted into two triangles).](/images/ray-triangle/triangulated.png)

A triangle is defined by three vertices (or points) whose positions are specified in three-dimensional space (Figure 1). One point can only represent a location in space. With two points, we can define a line. We can define a surface (and a plane) with three points. By construction, a triangle is coplanar; the three vertices represent a plane, and all three vertices lie in the same plane. This is not necessarily the case for polygons with more than three vertices; four points define a quad even when these four points are not in the same plane (which is possible). However, we can convert quads into two coplanar triangles, as depicted in Figure 2. This technique can be applied to a polygon with an arbitrary number of vertices: this process is called **triangulation** (we will learn about rendering polygon meshes in the next lesson).

## How Do We Compute the Intersection of a Ray With a Triangle?

Over the last few decades, several algorithms have been developed to compute the intersection between a ray and a triangle (research on this topic is still going and papers keep being published regularly). In this lesson, we will teach two of these techniques. The first technique requires just basic logic and elementary linear algebra to implement. The second technique, considered one of the fastest ray-triangle intersection algorithms, was proposed by Möller-Trumbore in 1997 in a paper titled "Fast, minimum storage ray-triangle intersection". Although it requires a more advanced understanding of linear algebra, we will do our best to work through it.

We can decompose the ray-triangle intersection test into a two steps problem:

- Does the ray intersect the plane defined by the triangle?
- If so, does the intersection point lie inside the triangle?

Let's see how we can answer these two questions.