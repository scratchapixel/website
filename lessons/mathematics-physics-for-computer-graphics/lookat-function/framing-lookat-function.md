<details>
In this short lesson, we will study a simple but useful method to place 3D cameras. To understand this lesson, you will need to be familiar with the concept of transformation matrix and cross-product between vectors. If that's not already the case, you might want to read the lesson [Geometry](/lessons/mathematics-physics-for-computer-graphics/geometry/) first.
</details>

## Placing the Camera

Being able to place the camera in a 3D scene is essential. However, in most of the lessons from Scratchapixel, we usually set the camera position and rotation (remember that scaling a camera doesn't make sense) using a 4x4 matrix which is often labeled the **camera-to-world** matrix. However, setting up a 4x4 matrix by hand is not friendly.

Thankfully, we can use a method that is commonly referred to as the **look-at** method. The idea is simple. To set a camera position and orientation, all you need is a point in space to set the camera position and a point to define what the camera is looking at (an aim). Let's label our first point "from" and our second point "to".

![](/images/lookat/look-at-1.png?)

We can easily create a world-to-camera 4x4 matrix from these two points as we will demonstrate in this lesson.

Before we get any further, however, let's address an issue that can be a source of confusion. Remember that in a right-hand coordinate system, if you are looking along the z-axis, the x-axis is pointing to the right, the y-axis is pointing upward and the z-axis is pointing towards you as shown in the figure below.

![](/images/lookat/look-at-setup.png?)

Therefore quite naturally, when we think of creating a new camera, it feels normal to orient the camera as if we were looking at the right-hand coordinate system with the z-axis pointing towards the camera (as shown in the image above). Because by convention cameras are oriented that way, books (e.g. Physically Based Rendering / PBRT) sometimes suggest that this is because cameras are not defined in a right-hand coordinate system but a left-hand one. If you look down the z-axis, a left-hand coordinate system is one in which the z-axis points away from you (in the same direction as the line of sight). Assuming the right-hand coordinate is the rule, why should we make an exception for cameras? This explanation is not inaccurate as such, it is nonetheless potentially a source of confusion.

We prefer to say that cameras are using a right-hand coordinate system like all the other objects in our 3D application. However, we do flip the orientation of the camera at render time, by "scaling" the ray direction by -1 along the camera's local coordinate z-axis when we cast rays into the scene. If you check the lesson [Ray-Tracing: Generating Camera Rays](/lessons/3d-basic-rendering/ray-tracing-generating-camera-rays/generating-camera-rays) you will notice that the ray-direction z-component is set to -1 before the ray direction vector is itself transformed by the camera-to-world matrix. This is not _stricto sensu_ a scaling. We just flip the direction of the ray direction vector along the camera's local coordinate system z-axis.

![](/images/lookat/look-at-setup1.png?)

!!!
Bottom line: if you use a right-hand coordinate system for your application, to keep things consistent, the camera should also be defined in a right-hand coordinate system like with any other 3D object. But as we cast rays in the opposite direction, it is as if the camera was indeed looking down along the negative z-axis. With this clarification out of the way, let's now see how we build this matrix.
!!!

![Figure 1: the local coordinate system of the camera aimed at a point.](/images/lookat/look-at-2.png?)

![Figure 2: computing the forward vector from the position of the camera and target point.](/images/lookat/look-at-3.png?)

Remember that a 4x4 matrix encodes the three axes of a Cartesian coordinate system. If this is not obvious to you, please read the lesson on [Geometry](/lessons/mathematics-physics-for-computer-graphics/geometry/). Remember that there are two conventions you need to pay attention to when you deal with matrices and coordinate systems. For matrices, you need to choose between row-major and column-major representations. Let's use the **row-major** notation. As for the coordinate system, you need to choose between the right-hand and the left-hand coordinate systems. Let's use a **right-hand** coordinate system. The fourth row of the 4x4 matrix (in a row-major matrix representation) encodes translation values.

$$
\begin{matrix}
\color{red}{Right_x}&\color{red}{Right_y}&\color{red}{Right_z}&0\\
\color{green}{Up_x}&\color{green}{Up_y}&\color{green}{Up_z}&0\\
\color{blue}{Forward_x}&\color{blue}{Forward_y}&\color{blue}{Forward_z}&0\\
T_x&T_y&T_z&1
\end{matrix}
$$

![Figure 3: the vector (0,1,0) is in the plane defined by the forward and up vector. The vector perpendicular to this plane is thus the right vector.](/images/lookat/look-at-4.png?)

How you name the axis of a Cartesian coordinate system is entirely up to you. You can call them x, y and z but in this lesson for clarity, we will name them **right** (for the x-axis), **up** (for the y-axis) and **forward** for the (z-axis). This is illustrated in figure 1. The method of building a 4x4 matrix from the from-to pair of points can be broken down into four steps:

- **Step 1: Compute the forward axis**. In Figures 1 and 2, it is quite easy to see that the forward axis of the camera's local coordinate system is aligned along the line segment defined by the points _from_ and _to_. A little bit of geometry suffices to calculate this vector. You just need to normalize the vector \(\text{From-To}\). Mind the direction of this vector: it is \(\text{From-To}\) not \(\text{To-From}\)). This can be done with the following code snippet:

  ```
  Vec3f forward = Normalize(From - to);
  ```

  Let's now calculate the over two vectors.

- **Step 2: Compute the right vector.** Recall from the lesson on Geometry that Cartesian coordinates are defined by three unit vectors that are perpendicular to each other. We also know that if we take two vectors \(A\) and \(B\), they can be seen as lying in a plane. Furthermore, the cross product of these two vectors creates a third vector \(C\) perpendicular to that plane and thus perpendicular to both \(A\) and \(B\). We can use this property to create the right vector. The idea here is to use some _arbitrary vector_ and calculate the cross vector between the forward vector and this arbitrary vector. The result is a vector that is necessarily perpendicular to the forward vector and that can be used in the construction of our Cartesian coordinate system as the right vector. The code for computing this vector is simple since it only implies a cross-product between the forward vector and this arbitrary vector:

  ```
  Vec3f right = crossProduct(randomVec, forward);
  ```

  How do we choose this _arbitrary_ vector? Well, this vector can't be arbitrary which is the reason why we wrote the word in italic. Think about this: if the forward vector is (0,0,1), then the right vector ought to be (1,0,0). This can only be done if we choose as our arbitrary vector, the vector (0,1,0). Indeed: <span class="code-inline">(0,1,0) x (0,0,1) = (1,0,0)</span> where the sign <span class="code-inline">x</span> here accounts for the cross product. Remember that the code/equation to compute the cross-product is: $$ \begin{array}{l} c_x = a_y * b_z - a_z * b_y,\\ c_y = a_z * b_x - a_x * b_z,\\ c_z = a_x * b_y - a_y * b_x\\ \end{array} $$ where \(a\) and \(b\) are two vectors and \(c\) is the result of the cross product of \(a\) and \(b\). When you look at figure 3, you can also notice that regardless of the forward vector's direction, the vector perpendicular to the plane defined by the forward vector and the vector (0,1,0) is always the right vector of the camera's Cartesian coordinate system. That's great because the vector (0,1,0) can be used as our _arbitrary vector_ (for now).

  <details>
  Note also from that observation that the right vector always lies in the xz-plane. How come you may ask? If the camera has a roll wouldn't the right vector be in a different plane? That's true, but applying a roll to the camera is not something you can do directly with the look-at method. To add a camera roll, you would first need to create a matrix to roll the camera (rotate the camera around the z-axis) and then multiply this matrix by the camera-to-world matrix built with the look-at method.
  </details>

  Finally, here is the code to compute the right vector:

  ```
  Vec3f tmp(0, 1, 0); Vec3f right = crossProduct(tmp, forward);
  ```

  Pay attention to the order of the vectors in the cross-product. Keep in mind that the cross-product is not commutative (it is anti-commutative, check the lesson on Geometry for more details). The best mnemonic way of remembering the right order is to think of the cross product of the forward vector (0,0,1) with the up vector (0,1,0) we know it should give (1,0,0) and not (-1,0,0). If you know the equations of the cross-product, you should easily find out that the order is \(up \times forward\) and not the other way around. Great we have the forward and right vectors. Let's find the "true" up vector.

- **Step 3: Compute the up vector.** Well this is very simple, we have two orthogonal vectors, the forward and right vector, so computing the cross product between these two vectors will just give us the missing third vector, the up vector. Note that if the forward and right vector is normalized, then the resulting up vector computed from the cross product will be normalized too (The magnitude of the cross product of u and v is equal to the area of the parallelogram determined by u and v \(\|u \times v\| = \|u\| \cdot \|v\| \cdot \sin \theta\)):

  ```
  Vec3f up = crossProduct(forward, right);
  ```

  Here again, you need to be careful about the order of the vectors involved in the cross-product. Great, we now have the three vectors defining the camera coordinate system. Let's now build our final 4x4 camera-to-world matrix.

- **Step 4: set the 4x4 matrix using the right, up, and forward vector as from point.** All there is to do to complete the process is to build the camera-to-world matrix itself. For that, we just replace each row of the matrix with the right data:
  - Row 1: replace the first three coefficients of the row with the coordinates of the _right_ vector,
  - Row 2: replace the first three coefficients of the row with the coordinates of the _up_ vector,
  - Row 3: replace the first three coefficients of the row with the coordinates of the _forward_ vector,
  - Row 4: replace the first three coefficients of the row with the coordinates of the _from_ point.

Again, if you are unsure about why we do that, check the lesson on Geometry. Finally here is the source code of the complete function. It computes and returns a camera-to-world matrix from two arguments, the _from_ and _to_ points. Note that the function's third parameter (_up_) is the _arbitrary_ vector used in the computation of the right vector. It is set to (0,1,0) in the main function but you may have to normalize it for safety (in case a user would input a non-normalized vector).

```
#include <cmath> 
#include <cstdint> 
#include <iostream> 
 
struct float3 
{ 
public: 
    float x{ 0 }, y{ 0 } , z{ 0 }; 
    float3 operator - (const float3& v) const 
    { return float3{ x - v.x, y - v.y, z - v.z }; } 
}; 
 
void normalize(float3& v) 
{ 
    float len = std::sqrt(v.x * v.x + v.y * v.y + v.z * v.z); 
    std::cout << v.x << " " << v.y << " " << v.z << "\n"; 
    v.x /= len, v.y /= len, v.z /= len; 
    std::cout << v.x << " " << v.y << " " << v.z << "\n"; 
} 
 
float3 cross(const float3& a, const float3& b) 
{ 
    return { 
        a.y * b.z - a.z * b.y, 
        a.z * b.x - a.x * b.z, 
        a.x * b.y - a.y * b.x 
    }; 
} 
 
struct mat4 
{ 
public: 
    float m[4][4] = {{1, 0, 0, 0},  {0, 1, 0, 0}, {0, 0, 1, 0}, {0, 0, 0, 1}}; 
    float* operator [] (uint8_t i) { return m[i]; } 
    const float* operator [] (uint8_t i) const { return m[i]; } 
    friend std::ostream& operator << (std::ostream& os, const mat4& m) 
    { 
        return os << m[0][0] << ", " << m[0][1] << ", " << m[0][2] << ", " << m[0][3] << ", " 
                  << m[1][0] << ", " << m[1][1] << ", " << m[1][2] << ", " << m[1][3] << ", " 
                  << m[2][0] << ", " << m[2][1] << ", " << m[2][2] << ", " << m[2][3] << ", " 
                  << m[3][0] << ", " << m[3][1] << ", " << m[3][2] << ", " << m[3][3]; 
    } 
 
}; 
 
void lookat(const float3& from, const float3& to, const float3& up, mat4& m) 
{ 
    float3 forward = from - to; 
    normalize(forward); 
    float3 right = cross(up, forward); 
    normalize(right); 
    float3 newup = cross(forward, right); 
 
    m[0][0] = right.x,   m[0][1] = right.y,   m[0][2] = right.z; 
    m[1][0] = newup.x,   m[1][1] = newup.y,   m[1][2] = newup.z; 
    m[2][0] = forward.x, m[2][1] = forward.y, m[2][2] = forward.z; 
    m[3][0] = from.x,    m[3][1] = from.y,    m[3][2] = from.z; 
} 
 
 
int main() 
{ 
    mat4 m; 
 
    float3 up{ 0, 1, 0 }; 
    float3 from{ 1, 1, 1 }; 
    float3 to{ 0, 0, 0 }; 
 
    lookat(from, to, up, m); 
 
    std::cout << m << std::endl; 
 
    return 0; 
} 
```

Should produce:

```
0.707107, 0, -0.707107, 0, -0.408248, 0.816497, -0.408248, 0, 0.57735, 0.57735, 0.57735, 0, 1, 1, 1, 1
```

## The Look-At Method Limitations

The method is very simple and works generally well. Though it has a weakness. When the camera is vertical looking straight down or straight up, the forward axis gets very close to the arbitrary axis used to compute the right axis. The extreme case is of course when the forward axis and this arbitrary axis are perfectly parallel e.g. when the forward vector is either (0,1,0) or (0,-1,0). Unfortunately, in this particular case, the cross-product fails to produce a result for the right vector. There is no real solution to this problem. You can either detect this case and choose to set the vectors by hand (since you know what the configuration of the vectors should be anyway). A more elegant solution can be developed using quaternion interpolation.