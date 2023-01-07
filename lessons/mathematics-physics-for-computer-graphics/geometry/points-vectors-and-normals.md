> Geometry is a branch of mathematics concerned with questions of shape, size, the relative position of figures, and the properties of space.

## A Word of Warning

This lesson will be long and tedious for most readers. If you are new to computer graphics, take the time to read it carefully. Understanding this part of the CG pipeline is crucial and will save you much time later.

## Introduction to Geometry

Points, vectors, matrices, and normals are to computer graphics what the alphabet is to literature; hence most CG books start with a chapter on linear algebra and geometry. However, for many looking to learn graphics programming, presenting a lot of maths before learning about making images can be pretty upsetting. So if you don't think CG programming is for you because you do not feel comfortable with maths or don't understand what a matrix is, don't give up now.

We began the "Foundation of 3D Rendering" section with a couple of lessons that do not require any prior knowledge of linear algebra for a reason. While this is a relatively unconventional way of teaching CG programming techniques, it's more exciting for you to get started with something practical and fun: an introductory ray tracer that requires very little knowledge of maths and some programming knowledge. Writing a renderer is a much more exciting and rewarding way of learning maths, as you can see incrementally how certain things are used to produce a concrete result (i.e., your final image). That being said, points, vectors, and matrices are instrumental in making CG images; we will use them extensively in every lesson.

In this lesson, you will learn what these constructs are, how they work, and the various techniques that can be used to manipulate them. This lesson will also explain the different conventions in linear algebra that CG researchers have used over the years when solving their problems and writing their code. You need to be aware of these conventions as they are often not mentioned in books (and poorly documented on the web). However, these conventions are essential; before reading or using another developer's code or techniques, you must check their conventions.

One quick note before we begin. If you are a mathematical purist, you might find it strange to see things explained here that are not technically related to linear algebra. We want to keep the scope of this lesson broad and include simple mathematical techniques commonly used in CG, which may only loosely relate to vectors and matrices. For instance, mathematically speaking, a point has nothing to do with linear algebra (a branch of mathematics only concerned with vectors). We chose to cover points because they are ubiquitous in CG (and the same mathematical techniques from linear algebra can be used to manipulate them). If you still need to understand the distinction between points and vectors, do not worry. We will cover that extensively in this chapter.

## What is Linear Algebra? Introduction to Vectors

So what exactly is linear algebra, and what will we study in this lesson? As we mentioned in the previous section, linear algebra is a branch of mathematics that has to do with the study of **vectors**. Now you might ask, "What is a vector, and how is it useful in the CG world?" We won't get into much detail, but a vector can be represented as an array of **numbers**. This array of numbers, which can assume any desired length, is also sometimes called a **tuple** in mathematics. If we want to be specific about the size of the vector, we may choose to say **n-tuple** where **n** represents the number of elements in the vector. Below is an example of the mathematical notation for a vector with 6 elements:

$$V = (a, b, c, d, e, f).$$

Where a, b, c, d, e, f are real numbers (1, 3, 4.56, -11, -13.08, 0, etc.).

The idea behind grouping these numbers is that collectively they represent another value or concept that is meaningful in the context of the problem. For example, in computer graphics, vectors can represent either a position or direction in space. We will also be able to transform (or modify) these vectors through a very powerful and compact series of operations. The process of transforming the content of a vector is achieved through what is called a **linear transformation**. We will spend much more time discussing transformations in a later section; for now, it is only necessary to recognize that they are instrumental.

## Points and Vectors

The terms **point** and **vector** are used in several scientific fields. In this lesson, we will explain their meaning in the context of computer graphics.

A **point** is a **position** in a three-dimensional space. A **vector**, on the other hand, usually means a **direction** (and some corresponding magnitude or size) in three-dimensional space. Vectors can be thought of as arrows pointing in various directions. Three-dimensional **points** and **vectors** are similar in that they are both represented by the tuple notation mentioned above.

$$V = (x, y, x).$$

Again, where (x, y, z) are real numbers.

![Figure 1: a point describes a position in space. A vector can be seen as a direction.](/images/geometry/pointvec.png?)

Remember, when talking to a mathematician or a physicist, their understanding of a vector or point could be far more general; they are not necessarily restricted to the use we make of them in CG. For them, a vector could be arbitrary or even infinite (meaning it can contain as many numbers as desired).

We will finish this chapter by briefly mentioning **homogeneous points**. Sometimes it is necessary to add a fourth element for mathematical convenience. An example of a point with homogeneous coordinates is given below:

$$P_H=(x, y, z, w).$$

Homogeneous points are used when it comes to multiplying points with matrices. Don't worry too much about them at this point in the lesson. We mention them now as they sometimes appear in the literature and can confuse readers. They will be explained in detail later in this lesson.

## A Quick Introduction to Transformations

You might still wonder how a linear transformation affects points and vectors. It's pretty simple. One of the most common operations we perform on points in CG consists of simply moving them around in space. This transformation is called **translation** and plays a vital role in rendering.

The translation operator is a linear transformation of the original point (which can be viewed as an input position point). Translation has no meaning when applied to a vector (which, remember, is a direction). This is because where the vector begins (that is, where it is centered) is unimportant; regardless of position, all "arrows" of the same length, pointing in the same direction, are equivalent. Instead, we very commonly use another linear transformation on vectors: rotation. Many more common operators can be used, but let's consider translation for points and rotations for vectors.

$$
\begin{array}{l}
P \rightarrow Translate \rightarrow P_T\\  
V \rightarrow Rotate \rightarrow V_T
\end{array}
$$

The subscripted letter \(\small T\) stands for "transformed".

As you may have noticed, we have yet to discuss what a vector's length, or magnitude, means. Indeed, the length of the arrow has great importance in CG. When the length of a vector is precisely 1, we say that the vector is **normalized** (you will hear and read this term all the time). Normalizing a vector involves altering the vector such that its length becomes 1, but its direction remains unchanged. Most of the time, we will want our vectors to be normalized. However, in some cases, not normalizing them can be preferred as the length of the vector will be meaningful.

For instance, imagine that you trace a line from point \(A\) to point \(B\). The line created is a vector that indicates where point \(B\) is located relative to point \(A\). It gives the direction of \(B\) as if you were standing at point \(A\). In this case, the vector's length indicates the distance from \(A\) to \(B\). This distance is sometimes required in specific algorithms.

Normalization of vectors is often a source of bugs in applications. Therefore, every time you declare a vector (or even use one), we recommend you consciously ask yourself if this vector is/isn't or should/shouldn't be normalized.

## Normals

![Figure 2: a normal is perpendicular to the plane tangent a P.](/images/geometry/normal.png?)

A normal is a technical term used in Computer Graphics (and Geometry) to describe the orientation of a surface of a geometric object at a point on that surface. Technically, the **surface normal** to a surface at point \(P\), can be seen as the vector perpendicular to a plane tangent to the surface at \(P\). Normals play an essential role in shading, where they are used to compute the brightness of objects (see further lessons on Lights and Shading).

Normals can be thought of as vectors with one caveat: they do not transform the same way as vectors. This is one of the main reasons we take the time to differentiate them. You will find more information on this topic in the chapter [Transforming Normals](/lessons/mathematics-physics-for-computer-graphics/geometry/transforming-normals). For now, it is only essential to understand what they are.

## From Theory to C++

In our C++ code, we won't distinguish between points, vectors, and normals; we represent all three with a Vec3 class (a template class so that we can create float, int, or double versions as needed). Some developers prefer to differentiate them. This limits the possibility of making mistakes. From experience, we found it more efficient (less code to write in the first place) to deal with one unique class (as the OpenEXR library does). However, we will still have to call a few specific functions carefully depending on whether or not the `Vec3` we are dealing with represents a point, a vector, or a normal. As you may remember, this is particularly critical when we use transformations. The full source code is provided in the download section of this lesson.

```
template<typename T> 
class Vec3 
{ 
public: 
    // 3 most basic ways of initializing a vector
    Vec3() : x(T(0)), y(T(0)), z(T(0)) {} 
    Vec3(const T &xx) : x(xx), y(xx), z(xx) {} 
    Vec3(T xx, T yy, T zz) : x(xx), y(yy), z(zz) {} 
    T x, y, z; 
}; 
 
typedef Vec3<float> Vec3f; 
 
Vec3<float> a; 
Vec3f b;
```

## Summary

From this first chapter, you should remember that mathematically a vector can be of any dimension. However, in CG, we use a more specific definition: a **vector** is a direction in 3D space (and therefore represented by three numbers). Additionally, we talk of **points** as representations of positions (also in 3D space and represented by three numbers). **Homogeneous points** are represented with four numbers but are a particular case we will study later.

Points and vectors can be transformed using **linear transformations**.

<details>
You will see the term linear transformation being used often. For example, if lines are preserved while being transformed, then we speak of a linear transformation (multiplication by a matrix is a linear transformation).
</details>

Typical examples of such transformations are **translation** for points and **rotation** for vectors. The length of a vector can be set to 1, in which case we say that it is **normalized**. The length of a vector (before it is normalized) represents the distance between two points and is sometimes needed in specific algorithms. For this reason, developers must be careful when and why they potentially choose to normalize a vector.

## What's Next?

One important thing we still need to explain is what the three numbers defining points and vectors represent. These numbers represent the coordinates of a point (in 2D or 3D space) with respect to a reference (also sometimes called the origin). This reference, which we technically call a **coordinate system**, is the topic of our next chapter.