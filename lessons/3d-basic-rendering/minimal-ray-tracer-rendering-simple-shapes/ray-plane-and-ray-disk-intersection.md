## Ray-Plane Intersection

![Figure 1: ray-plane intersection.](/images/ray-simple-shapes/plane.png?)

In this chapter, we will learn how to calculate the intersection of a ray with a plane and a disk. We know from the lesson on Geometry that the dot (or scalar) product of two vectors that are perpendicular to each other is always equal to 0:

$$A \cdot B = 0$$

Again this is only true if A and B are perpendicular. A plane can be defined as a point representing how far the plane is from the world's origin and a normal (defining the orientation of the plane). Let's call this point \(p_0\) and the plane normal \(n\). A vector can be calculated from any point on the plane by subtracting \(p_0\) from this point which we will call \(p\). Since the vector resulting from this subtraction lies in the plane, it should be perpendicular to the plane's normal, thus using the property that the dot product of two perpendicular vectors is equal to 0, we can write (equation 1):

$$(p - p_0) \cdot n = 0$$

Similarly, a ray can be defined using the following parametric form (equation 2):

$$l_0 + l * t = p$$

where \(l_0\) is the origin of the ray and \(l\) is the ray direction. This only means that we can calculate the position of any point along the ray from the ray's origin, its direction, and the term \(t\), which is a positive real number (which, as usual, is the parametric distance from the origin of the ray to the point of interest along the ray). If the ray and the plane intersect, they share a point where the line intersects the plane. If this point is \(p\), we can insert equation 2 in equation 1, and we get:

$$(l_0 + l * t - p_0) \cdot n = 0 $$

We are interested in computing a value for \(t\) from which we can calculate the position of this intersection point using the ray parametric equation. Solving for \(t\) we get:

$$
\begin{array}{l}
l * t \cdot n + (l_0 - p_o) \cdot n = 0\\
t = -{\dfrac{(l_0-p_0)\cdot n}{l \cdot n}} = {\dfrac{(p_0 - l_0) \cdot n}{l \cdot n}}
\end{array}
$$

Note that the plane and the ray are parallel when the denominator (the term \(l \cdot n\) gets close to 0\. Then, either the plane and the ray perfectly coincide, in which case there is an infinity of solutions, or the ray is away from the plane, where there is no intersection. Generally, in a C++ implementation, when the denominator is lower than a very small value, we return false (no intersection was found).

```
bool intersectPlane(const Vec3f &n, const Vec3f &p0, const Vec3f &l0, const Vec3f &l, float &t)
{
    // assuming vectors are all normalized
    float denom = dotProduct(n, l);
    if (denom > 1e-6) {
        Vec3f p0l0 = p0 - l0;
        t = dotProduct(p0l0, n) / denom; 
        return (t >= 0);
    }

    return false;
}
```

## Ray-Disk Intersection

![Figure 2: ray-disk intersection.](/images/ray-simple-shapes/disk.png?)

The ray-disk intersection routine is very simple. A disk is generally defined by a position (the disk center's position), a normal, and a radius. First, we can test if the ray intersects the plane in which lies the disk. For the ray-plane intersection step, we can use the code we have developed for the ray-plane intersection test. If the ray intersects this plane, all we have to do is calculate the intersection point and the distance from this point to this disk's center. The ray intersects the disk if this distance is lower or equal to the disk radius. Note that as an optimization, you can test the square of the distance against the square of the disk's radius. The square distance can be calculated from the dot product of this vector (v in the code) with itself. Technically, computing the distance would require taking the square root of this dot product. However, we can also test the result of this dot product directly against the square of the radius (which is generally precomputed) to avoid using an expensive square root operation.

```
bool intersectDisk(const Vec3f &n, const Vec3f &p0, const float &radius, const Vec3f &l0, const Vec3 &l)
{
    float t = 0;
    if (intersectPlane(n, p0, l0, l, t)) {
        Vec3f p = l0 + l * t;
        Vec3f v = p - p0;
        float d2 = dot(v, v);
        return (sqrtf(d2) &lt;= radius);
        // or you can use the following optimization (and precompute radius^2)
        // return d2 &lt;= radius2; // where radius2 = radius * radius
     }

     return false;
}
```