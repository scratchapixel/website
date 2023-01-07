## Improving the Rasterization Algorithm

![Figure 1: jagged edges pixel artifacts can be reduced using anti-aliasing.](/images/rasterization/jaggies.png)

![Figure 2: when one sample is used the triangle is missed. But by using sub-pixels, we can detect that the pixel overlaps the triangle at least partially. The pixel color is equal to the sum of the sub-pixel colors divided by the total number of sub-pixels or samples (in this example, 16 samples).](/images/rasterization/antialiasing3.png?)

![Figure 3: pixels can't properly capture the shape of continuous surfaces.](/images/rasterization/antialiasing1.png?)

![Figure 4: using sub-pixels to improve fight aliasing.](/images/rasterization/antialiasing2.png?)

![Figure 5: anti-aliasing helps with smoothing jagged edges.](/images/rasterization/antialiasing4.png?)

![Figure 6: 1 sample per pixel or 1 pps (top) vs. 4 samples per pixel (bottom).](/images/rasterization/antialiasing.png?)

![Figure 7: if the 4 corners of an 8x8 pixel grid overlap the triangle then all remaining pixels of the grids cover the triangle too.](/images/rasterization/rasterization-optimization.png?)

### Aliasing and Anti-Aliasing

All the techniques we presented in the previous chapters are the foundation of the rasterization algorithm. Though, we have only implemented these techniques in a very basic way. The GPU rendering pipeline and other rasterization-based production renderers use the same concepts but they used highly optimized versions of these algorithms. Presenting all the different tricks that are used to speed up the algorithm goes way beyond the scope of an introduction. We will just review some of them quickly now but we plan to devote a lesson to this topic in the future.

First, let's consider one basic problem with 3D rendering. If you zoom up on the image of the triangle that we rendered in the previous chapter, you will notice that the edges of the triangle are not regular (in fact, it is not specific to the edge of the triangle, you can also see that the checkerboard pattern is irregular on the edge of the squares). The steps that you can easily see in figure 1, are called **jaggies**. These jagged edges or stair-stepped edges (depending on what you prefer to call them) are not an artifact. This is just the result of the fact that the triangle is broken down into pixels. What we do with the rasterization process is break down a continuous surface (the triangle) into discrete elements (the pixels), a process that we already mentioned in the [Introduction to rendering](/lessons/3d-basic-rendering/rendering-3d-scene-overview/). The problem is similar to trying to represent a continuous curve or surface with Lego bricks. You simply can't, you will always see the bricks (figure 2). The solution to this problem in rendering is called **anti-aliasing** (also denoted AA). Rather than rendering only 1 sample per pixel (check if the pixel overlaps the triangle by testing if the point in the center of the pixel covers the triangles), we split the pixel into sub-pixels and repeat the coverage test for each sub-pixels. Of course, each sub-pixel is nothing else than another brick thus this doesn't solve the problem entirely. Nonetheless, it allows for capturing the edges of the objects with slightly more precision. Pixels are most of the time divided into an N by N number of sub-pixels where N is generally a power of 2 (2, 4, 8, etc.), though it can technically take on any value greater or equal to 1 (1, 2, 3, 4, 5, etc.). There are different ways of addressing this aliasing issue. The method we described belongs to the category of sampling-based anti-aliasing methods.

The pixel final color is computed as the sum of all the sub-pixels colors divided by the total number of sub-pixels. Let's take an example (as with pixels, if a sub-pixel or sample covers a triangle, it then takes on the color of that triangle, otherwise it takes on the background color which is usually black). Imagine that the triangle is white. If only 2 or 4 samples overlap the triangle, the final pixel color will be equal to (0+0+1+1)/4=0.5. The pixel won't be completely white, but it won't be completely black either. Thus rather than having a "binary" transition between the edge of the triangle and the background, that transition is more gradual, which reduces the stair-stepped pixels artifact. This is what we call **anti-aliasing**. To understand anti-aliasing you need to study signal processing theory, which again is a very large and pretty complex topic on its own.

The reason why it is best to choose N as a power of 2, is because most processors these days, can run several instructions in parallel and the number of instructions that run in parallel is also generally a power of 2. You can look on the Web for things such as SSE instruction sets which are specific to CPUs, but GPUs use the same concept. SSE is a feature that is available on most modern CPUs and that can be used to run generally 4 or 8 floating point calculations at the same (in one cycle). All this means is that, for the price of 1 floating point operation, you get 3 or 7 for free. This can in theory speed up your rendering time by 4 or 8 (you can never reach that level of performance though because you need to pay a small penalty for setting these instructions up). You can use SSE instructions for example to render 2x2 sup-pixels for the cost of computing 1 pixel, and as a result, you get smoother edges for "free" (the stair-stepped edges are less visible).

### Rendering Blocks of Pixels

Another common technique to accelerate rasterization is to render blocks of pixels, but rather than testing all pixels contained in the block, we first start to test pixels at the corners of the block. GPU algorithms can use blocks of 8x8 pixels. The technique used is more elaborate and is based on the concept of tiles, but we won't detail it here. If all four corners of that 8x8 grid cover the triangle, then necessarily, the other pixels of the grid also cover the rectangle (as shown in figure 7). In that case, there is no need to test all the other pixels which saves time. They can just be filled up with the triangle's colors. If vertex attributes need to be interpolated across the pixels block, this is also straightforward because if you have computed them at the block's corners, then all you need to do is linearly interpolate them in both directions (horizontally and vertically). This optimization only works when triangles are close to the screen and thus large in screen space. Small triangles don't benefit from this technique.

### Optimizing the Edge Function

The edge function too can be optimized. Let's have a look at the function implementation again:

```
int orient2d(const Point2D& a, const Point2D& b, const Point2D& c)
{
    return (b.x - a.x) * (c.y - a.y) - (b.y - a.y) * (c.x - a.x);
}
```

Recall that `a` and `b` in this function are the triangle vertices and that `c` is the pixel coordinates (in raster space). One interesting thing to note is that this function is going to be called for each pixel contained in the triangle bounding box. Though while we iterate over multiple pixels only `c` changes. The variables `a` and `b` stay the same. Suppose we evaluate the equation once and get a result `w0`:

```
w0 = (b.x - a.x) * (c.y - a.y) - (b.y - a.y) * (c.x - a.x);
```

Then `c.x` gets incremented by an amount `s` (the per pixel step). The new value of `w0` is going to be:

```
w0_new = (b.x - a.x) * (c.y - a.y) - (b.y - a.y) * (c.x + s - a.x);
```

Subtracting the first equation from the second, we get:

```
w0_new - w0 = -(b.y - a.y) * s;
```

The term `-(b.y - a.y) * s` is a constant value for a given triangle, because `s` is the same amount each time (one pixel), and `a` and `b` as already mentioned are constant too. We can calculate it once and store it in a variable (call it `w0_step`) and then the calculation reduces to:

```
w0_new = w0 + w0step;
```

You can do this for `w1` and `w2`, and also do a similar thing for the `c.y` step.

The edge function uses 2 mults and 5 subs but with this trick, it can be reduced to a simple addition (of course you need to compute the initial values). This technique is well documented on the internet. We won't be using it in this lesson but we will study it in more detail and implement it in another lesson devoted to advanced rasterization techniques.

### Fixed Point Coordinates

![Figure 8: fixed point coordinates.](/images/rasterization/subpixel-precision.png?)

Finally and to conclude this section, we will briefly talk about the technique that consists of converting vertex coordinates which are initially defined in floating-point format to fixed-point format just before the rasterization stage. Fixed-point is the fancy word (in fact the correct technical term) for integer. When vertex coordinates are converted from NDC to raster space, they are then also converted from floating point numbers to fixed point numbers. Why do we do that? There is no easy and quick answer to this question. But to be short, let's just say that GPUs use fixed point arithmetic because from a computing point of view, manipulating integers is easier and faster than manipulating floats or doubles (it only requires logical bit operations). Again this is just a very generic explanation. The conversion from floating point to integer coordinates and how is the rasterization process implemented using integer coordinates is a large and complex topic that is not documented on the Internet (you will find very little information about this topic which is very strange considering that this very process is central to the way modern GPUs work).

The conversion step involves rounding off the vertex coordinates to the nearest integer. Though if you only do so, then you sort of snap the vertex coordinates to the nearest pixel corner coordinates. This is not so much an issue when you render a still image, but it creates visual artifacts with animation (vertices are snapped to different pixels from frame to frame). The workaround is to convert the number to the smallest integer value but also reserve some bits to encode the **sub-pixel** position of the vertex (the fractional part of the vertex position). GPUs typically use 4 bits to encode sub-pixel precision (you can search graphics APIs documentation for the term sub-pixel precision). In other words, on a 32 bits integer, 1-bit might is used to encode the number's sign, 27 bits are used to encode the vertex integer position, and 4 bits are used to encode the fractional position of the vertex within the pixel. This means that the vertex position is "snapped" to the closest corner of a 16x16 sub-pixel grid as shown in figure 8 (with 4 bits, you can represent any integer number in the range [1:15]). Somehow the vertex position is still snapped to some grid corner, but snapping, in this case, is less of a problem than when the vertex is snapped to the pixel coordinates. This conversion process leads to many other issues one of which is **integer overflow** (overflow occurs when the result of an arithmetic operation produces a number that is greater than the number that can be encoded with the number of available bits). This may happen because integers cover a smaller range of values than floats. Things gets also sort of complex when anti-aliasing is thrown into the mix. It would take a lesson on its own to explore the topic in detail.

Fixed point coordinates allow to speed up the rasterization process and the edge function even more. This is one of the reasons for converting vertex coordinates to integers. These optimization techniques will be presented in another lesson.

## Notes Regarding our Implementation of the Rasterization Algorithm

Finally, we are going to quickly review the code provided in the source code chapter. Here is a description of its main components:

<details>
Use this code for learning purposes only. It is not efficient at all. Many simple optimizations could be made but we wrote it for clarity, not efficiency. Our goal is not to write production code but code that helps to learn how things work in principle. Optimizations are left to you but trying to improve the code could be a good exercise.
</details>

<details>
The object that is being rendered by the program is stored in an include file. This is fine for this small program but this would never be done in a professional application. Geometry files can be very large and including this data in the program means that the size of the program will become very large as well (which is not a good thing, plus compile time is going to increase a lot). Though again for this simple test and because the object we are using is quite light, this is not an issue. Though more generally if you are unsure about the way we represent the geometry of a 3D object in a program, check the lesson on this topic in the Modeling and Geometry section. The only information we will be using in this program are the triangle vertices' positions (in world space) and their texture or st coordinates.
</details>

- We will use the function `computeScreenCoordinates` to compute the screen coordinates using the method detailed in the lesson devoted to the [pinhole camera model](/lessons/3d-basic-rendering/3d-viewing-pinhole-camera/). This is essentially needed to be sure that our render output can be compared to a render in Maya which is also using a physically-based camera model.

  ```
  float t, b, l, r; 
  
  computeScreenCoordinates( 
      filmApertureWidth, filmApertureHeight, 
      imageWidth, imageHeight, 
      kOverscan, 
      nearClippingPLane, 
      focalLength, 
      t, b, l, r); 
  ```

- The function `convertToRaster()` is where we convert the triangle vertices coordinates from camera space to raster space. The function is similar to the function `computePixelCoordinates()` that we implemented in the previous lesson. Remember that in the second chapter of this lesson we learned about a method to convert coordinates from screen space to NDC space (keep in mind that in the GPU world, coordinates in NDC space are in the range [-1,1]). We will use the same remapping method in this function. The other important thing to remember is that up to the previous lesson, projected points were 2D points. From now on, they will need to be 3D points. In the x- and y-coordinates of these points, we will store the coordinates of the projected point in screen space. In the z-coordinate, we will store the vertices camera space z-coordinate. The z-coordinate is needed to solve the visibility problem as explained in [chapter four](/lessons/3d-basic-rendering/rasterization-practical-implementation/visibility-problem-depth-buffer-depth-interpolation) of this lesson.

  ```
  convertToRaster(v0, worldToCamera, l, r, t, b, nearClippingPLane, imageWidth, imageHeight, v0Raster); 
  convertToRaster(v1, worldToCamera, l, r, t, b, nearClippingPLane, imageWidth, imageHeight, v1Raster); 
  convertToRaster(v2, worldToCamera, l, r, t, b, nearClippingPLane, imageWidth, imageHeight, v2Raster);
  ```

- Don't forget that all vertex attributes associated with triangle vertices also need to be "pre-divided" by these vertices' z-coordinates (this is needed for perspective-correct interpolation). It is usually done just before the triangle gets rendered (just before the loop that iterates over the pixels). Though be careful because in the code the vertex z-coordinate is set to its reciprocal (to speed up the computation of the sample depth). Thus rather than using a division, we will use multiplication instead.

  ```
  // precompute reciprocal of the vertex z-coordinate
  v0Raster.z = 1 / v0Raster.z, 
  v1Raster.z = 1 / v1Raster.z, 
  v2Raster.z = 1 / v2Raster.z; 
  
  Vec2f st0 = st[stindices[i * 3]]; 
  Vec2f st1 = st[stindices[i * 3 + 1]]; 
  Vec2f st2 = st[stindices[i * 3 + 2]]; 
  
  // This is needed for perspective correct interpolation
  st0 *= v0Raster.z, st1 *= v1Raster.z, st2 *= v2Raster.z;
  ```

- The function contains two loops. One to iterate over all the triangles in the scene (the outer loop), and the second to iterate over all pixels contained in the bounding box overlapping the triangle that is being rendered (the inner loop). Note that, some variables in the inner loop are constant and can thus be pre-computed. This is the case for instance for the inverse of the triangle vertices' z-coordinates which are linearly interpolated for each pixel that covers the triangle (as well as the vertex attributes that need to be divided by their respective z-coordinates).

  ```
  // Outer loop. Loop over triangles
  for (uint32_t i = 0; i < ntris; ++i) {
      ...
      // Inner loop. Loop over pixels
      for (uint32_t y = y0; y <= y1; ++y) { 
          for (uint32_t x = x0; x <= x1; ++x) {
              ...
          }
      }
  }
  ```

- The rest of the program is straightforward. Each pixel is tested for coverage using the edge function technique. If a pixel covers a triangle, we compute the pixel barycentric coordinates. We then use these coordinates to compute the depth of the sample. If we pass the depth buffer test, we then update the buffer with the new depth value and update the frame-buffer with the triangle color.

  ```
  Vec3f pixelSample(x + 0.5, y + 0.5, 0); 
  float w0 = edgeFunction(v1Raster, v2Raster, pixelSample); 
  float w1 = edgeFunction(v2Raster, v0Raster, pixelSample); 
  float w2 = edgeFunction(v0Raster, v1Raster, pixelSample); 
  if (w0 >= 0 && w1 >= 0 && w2 >= 0) { 
      w0 /= area; 
      w1 /= area; 
      w2 /= area; 
      // linearly interpolate sample depth
      float oneOverZ = v0Raster.z * w0 + v1Raster.z * w1 + v2Raster.z * w2; 
      float z = 1 / oneOverZ; 
      // do we pass the depth buffer test?
      if (z < depthBuffer[y * imageWidth + x]) { 
          depthBuffer[y * imageWidth + x] = z;
          // update frame buffer
          ...
      }
  }
  ```

- **Shading wise:** to make it visually more interesting we will be using several shading techniques. The model we are using has only one vertex attribute: st or texture coordinates. The texture coordinates can be used to create a checkerboard pattern which is then combined with a simple shading trick called facing ratio. The facing ratio takes the dot product between the triangle normal (which we can compute with a simple cross product between any two edges of the triangle) and the view direction. The view direction is simply the vector defined by the point P on the triangle that is being shaded and the camera position E. Since all points at this point of the program is defined in camera space, the camera position (or the eye position) is simple E=(0,0,0). Thus the view direction can simply be computed as -P which then needs to be normalized. The dot product can be negative so we need to clamp it (we only want positive values).

  ```
  Vec3f n = (v1Cam - v0Cam).crossProduct(v2Cam - v0Cam); 
  n.normalize(); 
  Vec3f viewDirection = -pt; 
  viewDirection.normalize(); 
  // facing ratio 
  float nDotView =  std::max(0.f, n.dotProduct(viewDirection));
  ```
  
  Note that for this technique to work, we also need to find the coordinates of P. The position of the point on the triangle that the pixel overlaps can be computed like any vertex attribute. We need to take the vertices in camera space, divide them by their respective z-coordinate, interpolate them with the barycentric coordinates and multiply the result by the sample depth (which is also coincidentally P' z-coordinate):

  ```
  // Get triangle vertices in camera space.
  Vec3f v0Cam, v1Cam, v2Cam; 
  worldToCamera.multVecMatrix(v0, v0Cam); 
  worldToCamera.multVecMatrix(v1, v1Cam); 
  worldToCamera.multVecMatrix(v2, v2Cam); 
  
  // Divide them by the respective z-coordinate as with any other vertex attribute and interpolate using 
  // barycentric coordinates.
  float px = (v0Cam.x/-v0Cam.z) * w0 + (v1Cam.x/-v1Cam.z) * w1 + (v2Cam.x/-v2Cam.z) * w2; 
  float py = (v0Cam.y/-v0Cam.z) * w0 + (v1Cam.y/-v1Cam.z) * w1 + (v2Cam.y/-v2Cam.z) * w2; 
   
  // P in camera space
  Vec3f pt(px * z, py * z, -z);
  ```
  
  Don't worry if you don't understand these shading techniques very well. They will be studied as soon as we get to the lessons on shading.

- Finally, the content of the frame buffer is stored in a PPM file.

Here is the result the program should produce. On the left, you can see a render of the object in Maya. On the right, is our render. As you can see the results are identical in terms of objects' position and shading. Maya uses a technique called stochastic sampling that has for effect to make the aliasing artifact slightly less visible than in our render, but the difference is subtle.

![](/images/rasterization/cowresult.png?)

As you can see, there is nothing magic about rendering. When you know the rules, you can reproduce images produced by professional applications.

![](/images/rasterization/cowresult1.png?)

![](/images/rasterization/cowresult2.png?)

As a bonus, we exported the position in the world space of the point on the triangle that each pixel in the image overlaps. We then displayed all the points in a 3D viewer. You can see the results below. Not surprisingly, points only show up on parts of the object that are directly visible to the camera. This shows that the depth-buffer technique works as expected. Every triangle on the back of the object that is hidden by another triangle is not rendered. The second image shows a close-up of the same point set (right).

## Conclusion

The main advantage of rasterization is simplicity and speed. The main drawback is that it is only useful to solve the visibility problem. Remember that rendering involves two steps: visibility and shading. The algorithm is of no use when it comes to shading.

In this lesson, we have told you everything you needed to know about the rasterization algorithm. Everything else you can learn from there is not so much about the technique itself but more about optimizing it (how to run it in parallel, how to make it efficient, how to run it on the GPU, etc.).

We hope to have provided one of the best introductions to the rasterization technique. If you appreciated this work and learned something while reading this lesson, please consider donating.

## Exercises

- Implement anti-aliasing. Divide the pixel into 4 sub-pixels. Generate a sample in the middle of each sample, and run the coverage test for each sample. Sum up the color of the samples and divide by 4. Set the pixel color to that result.
- Implement back-face culling. Don't render the triangle if the dot product of its normal and the view direction is lower than a certain threshold. Remember that if two vectors point in opposite directions, their dot product is negative. Remember that the dot product of two vectors is the cosine of the angle between the two vectors. Express the threshold value as an angle in degrees.
- Write the content of the z-buffer to the PPM file (you will need to remap the values of the depth buffer to the range [0,255]).

## Reference

- A Parallel Algorithm for Polygon Rasterization. Juan Pineda. Siggraph 1988.
- A Subdivision Algorithm for Computer Display of Curved Surfaces. Edwin Earl Catmull. Thesis 1974.
- Fundamentals of Texture Mapping and Image Wrapping. Paul S. Heckbert. Thesis 1989.