We have covered everything there is to say! We are now prepared to write our first ray tracer. You should now be able to guess how the ray-tracing algorithm works.

First of all, take a moment to notice that the propagation of light in nature is just a countless number of rays emitted from light sources that bounce around until they hit the surface of our eye. Ray-tracing is, therefore, elegant in that it is based directly on what happens around us. Apart from the fact that it follows the path of light in reverse order, it is nothing less than a perfect nature simulator.

The ray-tracing algorithm takes an image made of pixels. For each pixel in the picture, it shoots a primary ray into the scene. The direction of that primary ray is obtained by tracing a line from the eye to the center of that pixel. Once we have that primary ray's direction set, we check every object in the scene to see if it intersects with any of them. In some cases, the primary ray will intersect more than one object. When that happens, we select the object whose intersection point is the closest to the eye. We then shoot a shadow ray from the intersection point to the light (Figure 1). 

![Figure 1: We shoot a primary ray through the center of the pixel to check for a possible object intersection. When we find one, we cast a shadow ray to determine if the point is illuminated or in shadow.](/images/introduction-to-ray-tracing/lightingnoshadow.gif)

The hit point is illuminated if this ray does not intersect an object on its way to the light. If it does intersect with another object, that object casts a shadow on it (Figure 2).

![Figure 2: The small sphere casts a shadow on the large sphere. The shadow ray intersects the small sphere before it gets to the light.](/images/introduction-to-ray-tracing/lightingshadow.gif)

If we repeat this operation for every pixel, we obtain a two-dimensional representation of our three-dimensional scene (Figure 3).

![Figure 3: To render a frame, we shoot a primary ray for each pixel of the frame buffer.](/images/introduction-to-ray-tracing/pixelrender.gif)

Here is an implementation of the algorithm in pseudocode:

```
for (int j = 0; j < imageHeight; ++j) { 
    for (int i = 0; i < imageWidth; ++i) { 
        // compute primary ray direction
        Ray primRay; 
        computePrimRay(i, j, &primRay); 
        // shoot prim ray in the scene and search for the intersection
        Point pHit; 
        Normal nHit; 
        float minDist = INFINITY; 
        Object object = NULL; 
        for (int k = 0; k < objects.size(); ++k) { 
            if (Intersect(objects[k], primRay, &pHit, &nHit)) { 
                float distance = Distance(eyePosition, pHit); 
                if (distance < minDistance) { 
                    object = objects[k]; 
                    minDistance = distance;  //update min distance 
                } 
            } 
        } 
        if (object != NULL) { 
            // compute illumination
            Ray shadowRay; 
            shadowRay.direction = lightPosition - pHit; 
            bool isShadow = false; 
            for (int k = 0; k < objects.size(); ++k) { 
                if (Intersect(objects[k], shadowRay)) { 
                    isInShadow = true; 
                    break; 
                } 
            } 
        } 
        if (!isInShadow) 
            pixels[i][j] = object->color * light.brightness; 
        else 
            pixels[i][j] = 0; 
    } 
} 
```

As one can see, the beauty of ray tracing is that it takes just a few lines to code; one could write a basic ray tracer in 200 lines. Unlike other algorithms, such as a scanline renderer, ray tracing takes little effort to implement.

Arthur Appel first described this technique in 1969 in a paper entitled "Some Techniques for Shading Machine Renderings of Solids". So, if this algorithm is so wonderful, why didn't it replace all the other rendering algorithms? The main reason, at the time (and even today to some extent), was speed. As Appel mentions in his paper:

> This method is very time consuming, usually requiring several thousand times as much calculation time for beneficial results as a wireframe drawing. About one-half of this time is devoted to determining the point-to-point correspondence of the projection and the scene.

In other words, it is slow (as Kajiya - one of the most influential researchers of all computer graphics history -once said: "ray tracing is not slow - computers are"). It is incredibly time-consuming to find the intersection between rays and geometry. For decades, the algorithm's speed has been the main drawback of ray tracing. However, as computers become faster, it is less and less of an issue. Although one thing must still be said: comparatively to other techniques, like the z-buffer algorithm, ray-tracing is still much slower. However, with fast computers today, we can compute a frame that used to take one hour in a few minutes or less. Real-time and interactive ray tracers are a hot topic.

To summarize, it is essential to remember that the rendering routine can be considered two separate processes. One step determines if a point at the surface of an object is visible from a particular pixel (the visibility part), and the second shades that point (the shading part). Unfortunately, both steps require expensive and time-consuming ray-geometry intersection tests. The algorithm is elegant and powerful but forces us to trade rendering time for accuracy and vice versa. Since Appel published his paper, much research has been done to accelerate the ray-object intersection routines. As computers became more powerful and combined with these acceleration techniques, ray-tracing became usable in everyday production environments and is, by today's standards, the defacto method used by most, if not all, rendering offline software. Video game engines are still using the rasterization algorithm. However, real-time ray-tracing is also in reach with the recent advent of GPU-accelerating ray-tracing (2017-2018) and RTX technology. While some video games already provide modes in which ray tracing can be turned on, it's only limited to simple effects such as sharp reflections and shadows.
