Now that we have explained the concept of a (cartesian) coordinate system (and how points' and vectors' coordinates relate to coordinate systems), we can look at some of the most common operations performed on points and vectors. This should cover the most common functions in any 3D application and renderer.

## Vector Class in C++

First, let's define our C++ Vector class:

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
```

## Vector Length

As mentioned in the previous paragraph, a vector can be seen as an arrow starting from one point and finishing at another. The vector itself indicates not only the direction of point B from A but can also be used to find the distance between A and B. This is given by the length of a vector which can easily be computed with the following formula:

$$||V|| = \sqrt{V.x * V.x + V.y * V.y + V.z * V.z}$$

In mathematics, the double bar (||V||) notation indicates the length of a vector. The **vector's length** is sometimes also called **norm** or **magnitude** (figure 1).

```
template<typename T> 
class Vec3 
{ 
public: 
    ... 
    // length can be a method from the class...
    T length() 
    { 
        return sqrt(x * x + y * y + z * z); 
    } 
    ... 
}; 
 
// ... or you can also compute the length in a function that is not part of the class
template<typename T> 
T length(const Vec3<T> &v) 
{ return sqrt(v.x * v.x + v.y * v.y + v.z * v.z); } 
```

Note that the axes of the three-dimensional cartesian coordinate systems are unit vectors.

## Normalizing a Vector

<details>
We sometimes use normalize with an 's' and normalize with a 'z'. We have mixed cultural influences, which is why we sometimes use one or the other. Still, in programming, though, the convention is often to use American spelling in the name of methods or functions, which is why it comes to writing code, we will always use normalize.
</details>

A normalized vector (we will use normalize with a z here which is the standard in the industry) is a vector whose length is 1 (vector B in figure 1). Such a vector is also called a **unit vector** (a vector with unit length). Normalizing a vector is very simple. We first compute the vector's length and divide each vector's coordinates by this length. The mathematical notation is:

$$ \hat{V} = {V \over { || V || }}$$

![Figure 1: the magnitude or length of vectors A and B is denoted by the double bar notation. A normalized vector is a vector whose length is 1 (in this example, vector B).](/images/geometry/normalize.png)

Note that the C++ implementation can be optimized. First, we only normalize the vector if its length is greater than 0 (as dividing by 0 is forbidden). We then compute a temporary variable, the invert of the vector length, and multiply each coordinate of the vector with this value rather than dividing them by the vector's length. As you may know, multiplications in a program are less costly than divisions. This optimization can be significant, as normalizing a vector is a widespread operation in a renderer that can be applied to thousands, hundreds of thousands, or millions of vectors (when not more). Of course, any possible optimization will impact the final render time at this level. However, some compilers will manage that for you under the hood. But you can always make that optimization explicit in your code.

```
template<typename T> 
class Vec3 
{ 
public: 
    ... 
    // as a method of the class Vec3
    Vec3<T>& normalize() 
    { 
        T len = length(); 
        if (len > 0) { 
            T invLen = 1 / len; 
            x *= invLen, y *= invLen, z *= invLen; 
        } 
 
        return *this; 
    } 
    ... 
}; 
 
// or as a utility function
template<typename T> 
void normalize(Vec3<T> &v) 
{ 
    T len2 = v.x * v.x + v.y * v.y + v.z * v.z; 
    // avoid division by 0
    if (len2 > 0) { 
        T invLen = 1 / sqrt(len2); 
        x *= invLen, y *= invLen, z *= invLen; 
    } 
} 
```

In mathematics, you will also find the term **norm** to define a function that assigns a length or size (or distance) to a vector. For example, the function we have just described is called the **Euclidean norm**.

## Dot Product

![Figure 2: the dot product of two vectors can be seen as the projection of A over B. if the two vectors A and B have unit length, then the result of the dot product is the cosine of the angle subtended by the two vectors.](/images/geometry/dotproduct.png)

The dot product or scalar product requires two vectors, A and B, and can be seen as the projection of one vector onto the other. The result of the dot product is a real number (a float or double in programming). A dot product between two vectors is denoted with the dot sign: \(A \cdot B\) (it can also be sometimes written as \(<A, B>\)). The dot product consists of multiplying each element of the A vector with its counterpart from vector B and taking the sum of each product. In the case of 3D vectors (length of the vector is three, they have three coefficients or elements which are x, y and z), it consists of the following operation:

$$A \cdot B = A.x * B.x + A.y * B.y + A.z * B.z$$

Note that this is quite similar to how we compute a vector's length (distance this time). If we take the square root (\(\sqrt{A \cdot B}\)) of the dot product between two equal vectors (A=B), then what we get is the length of the vector. We can write:

$$||V||^2=V \cdot V$$

It can be used in the normalized method:

```
template<typename T> 
class Vec3 
{ 
public: 
    ... 
    T dot(const Vec3<T> &v) const 
    { 
        return x * v.x + y * v.y + z * v.z; 
    } 
 
    Vec3<T>& normalize() 
    { 
        T len2 = dot(*this); 
        if (len2 > 0) { 
            T invLen = 1 / sqrt(len2); 
            x *= invLen, y *= invLen, z *= invLen; 
        } 
 
        return *this; 
    } 
    ... 
}; 
 
template<typename T> 
T dot(const Vec3<T> &a, const Vec3<T> &b) 
{ return a.x * b.x + a.y * b.y + a.z * b.z; } 
```

The dot product between two vectors is an essential and common operation in any 3D application because the result of this operation relates to the **cosine of the angle** between the two vectors. Figure 2 illustrates the geometric interpretation of the dot product. In this example, vector A is projected in the direction of vector B.

- If B is a unit vector, then the product \(A \cdot B\) gives \(||A||\cos(\theta)\), the magnitude of the projection of A in the direction of B, with a minus sign if the direction is opposite. This is called the scalar projection of A onto B.

- When neither A nor B is a unit vector, we can write that \(A \cdot { B / ||B|| } \) since B as a unit vector is \(B / ||B||\).

- When the two vectors are normalized, then taking the arc cosine of the dot product gives you the angle \(\theta\) between the two vectors: \(\theta = \cos^{-1}({{A \cdot B} / {||A||\:||B||}})\) or \(\theta=\cos^{-1}(\hat A \cdot \hat B)\) (in mathematics, \(\cos^{-1}\) is the inverse of the \(\cos\) function. In computer programming languages, this function is generally denoted `acos()`).

<details>
![](/images/geometry/dotproduct1.png)

The dot product is an essential operation in 3D. It can be used for many things, for example, as a test of orthogonality. When two vectors are perpendicular to each other (A.B), the result of the dot product between these two vectors is 0. When the two vectors point in opposite directions (A.C), the dot product returns -1. When they point in the same direction (A.D), it returns 1. It is also used intensively to find out the angle between two vectors or compute the angle between a vector and the axis of a coordinate system (which is useful when the coordinates of a vector are converted to spherical coordinates. This is explained in the chapter on [trigonometric functions](/lessons/mathematics-physics-for-computer-graphics/geometry/spherical-coordinates-and-trigonometric-functions)).
</details>

## Cross Product

The **cross product** is also an operation on two vectors, but to the difference of the dot product, which returns a number, the cross product returns a vector. The particularity of this operation is that the vector resulting from the cross product is perpendicular to the other two (this is shown in figure 3). The cross-product operation is written using the following syntax:

$$C = A \times B$$

![Figure 3: the cross product of two vectors, A and B, gives a vector C perpendicular to the plane defined by A and B. When A and B are orthogonal to each other (and have unit length), A, B, and C form a Cartesian coordinate system.](/images/geometry/crossproduct.png)

To compute the cross-product, we will need to implement the following formula:

$$
\begin{array}{l}
C_X = A_Y * B_Z - A_Z * B_Y\\
C_Y = A_Z * B_X - A_X * B_Z\\
C_Z = A_X * B_Y - A_Y * B_X
\end{array}
$$

The result of the cross-product is another vector that is orthogonal to the other two. A cross product between two vectors is denoted with the cross sign: \(A \times B\). The two vectors, A and B, define a plane, and the resulting vector, C, is perpendicular to that plane. Vectors A and B don't have to be perpendicular to each other, but when they are, the resulting A, B, and C vectors form a cartesian coordinate system (assuming the vectors have unit length). This is particularly useful for creating coordinate systems, which we will explain in the chapter [Creating a Local Coordinate System](/lessons/mathematics-physics-for-computer-graphics/geometry/creating-an-orientation-matrix-or-local-coordinate-system).

```
template<typename T> 
class Vec3 
{ 
public: 
    ... 
    // as a method of the class...
    Vec3<T> cross(const Vec3<T> &v) const 
    { 
        return Vec3<T>( 
            y * v.z - z * v.y, 
            z * v.x - x * v.z, 
            x * v.y - y * v.x); 
    } 
    ... 
}; 
 
// or as a utility function
template<typename T> 
Vec3<T> cross(const Vec3<T>  &a, const Vec3<T> &b) 
{ 
    return Vec3<T>( 
        a.y * b.z - a.z * b.y, 
        a.z * b.x - a.x * b.z, 
        a.x * b.y - a.y * b.x); 
} 
```

Suppose you need a mnemonic way of remembering this formula. In that case, we like to use the technique that consists of asking ourselves the question "why z?", y and z being the coordinates of vectors A and B used to compute the x coordinate of the resulting vector C (because indeed "why z?" - 'why' here, of course, stands for the letter 'y'). More seriously, logic can easily be used to reconstruct this formula. Since you know that the result of the cross product is a vector perpendicular to the other two, you know that if A and B are the x- and y-axis of a cartesian coordinate system, the cross product of A and B should give you the z-axis that is (0,0,1). The only way you can get this result is if Cz = 1, which is only true when Cz = A.x * B.y - A.y * B.x. You can deduce the other coordinates used to compute Cx and Cy. Finally, the easiest method might be to write the cross-product operation in the following form:

$$
\begin{pmatrix}
a_x\\a_y\\a_z
\end{pmatrix}
\times
\begin{pmatrix}
b_x\\b_y\\b_z
\end{pmatrix} = 
\begin{pmatrix}
a_yb_z - a_zb_y\\
a_zb_x - a_xb_z\\
a_xb_y - a_yb_x
\end{pmatrix}
$$

Presenting the vector in a column vector form shows that to find any coordinate of the resulting vector (for example, x), we need to use the other two (y and z if x is the coordinate we wish to compute) from vectors A and B.

It is essential to note that the order of the vectors involved in the cross-product affects the resulting vector C. If we take the previous example (taking the cross product between the x- and the y-axis of a cartesian coordinate system), you can see that A x B doesn't give you the same result as B x A:

$$A \times B = (1,0,0) \times (0,1,0) = (0,0,1),$$

whereas

$$B \times  A=(0,1,0) \times (1,0,0)=(0,0,-1).$$

![Figure 4: use your left or right hand to determine the orientation of vector C (the normal, for instance) when the index fingers point along A and the middle finger points along B.](/images/geometry/normalleftrighthand.png)

![Figure 5: using your right hand, you can align your index finger along either A or B and the middle finger against the other vector (B or A) to find out if C (the normal, for instance) points upwards or inwards in the right-hand coordinate system.](/images/geometry/normalleftrighthand2.png)

We say that the cross product is **anticommutative** (swapping the position of any two arguments negates the result): If AxB=C, then BxA=-C. Remember from the previous chapter that when two vectors are used to define the first two bases of a coordinate system, the third vector can point on either side of the plane. We also described how you use your hands to differentiate the two systems. When you compute a cross-product between vectors, you will always get the same unique solution. For instance, if A = (1, 0, 0) and B = (0, 1, 0), C can only be (0, 0, 1). So why should I care about the handedness of my coordinate system, then? Because if the result of the computation is always the same, the way you draw the resulting vector, however, depends on the handedness of your coordinate system. You can use the same mnemonic technique to determine which direction the vector should point to, depending on your convention. In the case of a right-hand coordinate system, if you align the index finger along the A vector (for example, the tangent at a point on the surface) and the middle finger along the B vector (the bitangent if you try to figure out the orientation of a normal), the thumb will point in the direction of the C vector (the normal). Note that if you use the same technique but with the left hand on the same vectors, A and B, your thumb will point in the opposite direction. Remember, though, that this is only a **representation** issue.

In mathematics, the result of a cross product is called a **pseudo vector**. The order of the vector in the cross-product operation is essential when **surface normals** are computed from the tangent and bitangent at the point where the normal is calculated. Depending on this order, the resulting normal can either be pointing towards the interior of the surface (**inward-pointing normal**) or away from it (**outward-pointing normal**). You can find more information on this topic in the chapter [Creating an Orientation Matrix](#).

## Vector/Point Addition and Subtraction

Other mathematical operations on points are usually straightforward. For example, a multiplication of a vector by a scalar or another vector gives a point. We can add two vectors to each other, subtract them, divide them, etc. Note that some 3D APIs distinguish between points, normals, and vectors. Technically, there are subtle differences between them, which can justify creating three separate C++ classes. For example, normals are not transformed like points and vectors (we will learn about that in this lesson), subtracting two points technically gives a vector, adding a vector to another vector or a point gives a point, etc. However, from practice, we found that writing these three C++ distinct classes to represent each type is not worth some of the complexity that comes with it. Similarly to OpenEXR, which has become an industry standard, we chose to describe all types with a single templated class called `Vec3`. We, therefore, make no distinction between normal, vector, and points (from a coding point of view). We will need to manage the (rare) exceptions when variables representing different types (normal, vector, points) but declared under the generic type `Vec3`, should be processed differently. Here is some C++ code to represent the most common operations (you will find the complete source code at the end of this lesson):

```
template<typename T> 
class Vec3 
{ 
public: 
    ... 
    Vec3<T> operator + (const Vec3<T> &v) const 
    { return Vec3<T>(x + v.x, y + v.y, z + v.z); } 
    Vec3<T> operator - (const Vec3<T> &v) const 
    { return Vec3<T>(x - v.x, y - v.y, z - v.z); } 
    Vec3<T> operator * (const T &r) const 
    { return Vec3<T>(x * r, y * r, z * r); } 
    ... 
}; 
```