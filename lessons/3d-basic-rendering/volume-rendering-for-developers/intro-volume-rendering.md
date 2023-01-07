## Preamble: Structure of this Lesson and Approach

Volume rendering (technically, using the term **participating media** instead of the term volume would be better) is a topic almost as large and as complex as hard surface rendering. It has its own set of equations which are in fact, almost a generalization of the equations used to describe how light interacts with hard matter. They can be overwhelming for readers that are not necessarily comfortable with such complex mathematical formulations.

True to the way we teach things on Scratchapixel, we chose a "bottom-up" approach to the challenge of teaching volume rendering. Or to say it differently, a practical approach. Rather than starting from the equations and diving into them, we will instead write code to render a simple volumetric sphere and explain things along the way in a hopefully intuitive way. Then everything we learned to that point will be summed up and formalized at the end of the lesson.

Several lessons will be devoted to volume rendering (it is a vast topic). In this introductory lesson, we will study the foundations of volume rendering and ray-marching. The next lessons will be devoted to other possible methods for rendering volumes, global illumination applied to participating medium, multiple scattering, formats used to store volume data such as OpenVDB, etc.

If you are interested in volume rendering, you might also want to check the lesson [Simulating the Colors of the Sky](/lessons/procedural-generation-virtual-worlds/simulating-sky/). The sky is a kind of volume.

## Introduction

Our aim in the first two chapters of this lesson is to learn how to render a volume shaped like a sphere illuminated by a single light source on a uniformly colored background. This will help us get a first intuitive understanding of what volumes are and introduce the ray-marching algorithm that we will be used to render them.

In this particular chapter, we will just render a plain volume with uniform density. We will ignore shadows cast by objects from outside or from within the volume as well as how to render volume with varying densities. These will be studied in the next chapters.

Instead of providing a lot of detailed background about what volumes are and the equations used to render them, let's dive straight into implementation and derive a more formal understanding of volume rendering from there.

![](/images/volume-rendering-developers/voldev-example.png?)

## Internal Transmittance, Absorption, Particle Density, and the Beer's Law

Light reaching our eyes as a result of light being reflected by an object or being emitted by a light source is likely to be absorbed as it travels through a volume of space filled up with some particles. The more particles we have in the volume, the more opaque the volume. From this simple observation, we can lay down some fundamental concepts related to volume rendering: absorption, transmission, and the relationship between how opaque a volume is and the density of particles it contains. For now, we will consider that the density of particles contained by the volume is uniform.

![](/images/volume-rendering-developers/voldev-L0term1.png?)

As light travels through the volume in the direction of our eye (which is how images of objects we are seeing are formed in our eye), some of it will be absorbed by the volume as it passes through it. This phenomenon is called **absorption**. What we are interested in (for now) is the amount of light that's being transmitted from the background through the volume. We speak of **internal transmittance** (the amount of light being absorbed by the volume as it travels through it). Internal transmittance can be seen as a value going from 0 (the volume blocks all light) to 1 (well, it's a vacuum so all light is transmitted).

The amount of light that's being transmitted through that volume is governed by the **Beer-Lambert law** (or Beer's law for short). In the Beer-Lambert law, the concept of density is expressed in terms of the absorption coefficient (and scattering coefficient but we will introduce the scattering coefficient later in this chapter). Which you can understand as, "the denser the volume, the higher the absorption coefficient"; and as you can guess intuitively, the volume gets more opaque as the absorption coefficient increases. The Beer-Lambert law equation looks like this:

$$
T = exp(-distance * \sigma_a)=e^{(-distance * \sigma_a)}
$$

The law states that there is an exponential dependence between the internal transmittance, \(T\), of light through a volume and the product of the **absorption coefficient** of the volume, \(\sigma_a\) (the Greek letter sigma), and the distance the light travels through the material (i.e. the path length).

The unit of these coefficients is a reciprocal distance or inverse length such as \(cm^{-1}\) or \(m^{-1}\) (this doesn't matter, it is just a matter of scale). This is important because it helps get an intuitive sense of what information these coefficients hold. You can consider the absorption coefficient (and the scattering coefficient that we will introduce later) as a probability or likelihood if you prefer that a random event occurs (e.g. the photon is absorbed or scattered) at any given point/distance.

The absorption and scattering coefficient are said to express a [probability density](https://en.wikipedia.org/wiki/Probability_density_function) (just in case you want to do more research on this topic). As it is a probability it should not exceed 1 however, this depends on the unit in which it is measured. For example, if you used millimeters you might get 0.2 for a given medium. But expressed in centimeter and meter this would become 2 and 20 respectively. So in practice, nothing stops you from using values greater than 1.

<details>
<summary>Relationship between the coefficients and the mean free path.</summary>
The fact that the unit of the absorption and scattering coefficients is inverse length is important because if you take the inverse of the coefficients (1 over the absorption of scattering coefficient) you get a distance. This distance called the mean free path represents the average distance at which a random event occurs:

$$
\text{mean free path} = { {1}\over{\sigma_s}}
$$ 

This value plays an important role in simulating multiple scattering in participating media. Check the lessons on subsurface scattering and advanced volume rendering to learn more about these very cool topics.
</details>

![Figure 1: the greater the distance or the greater the density the lower the internal transmittance value.](/images/volume-rendering-developers/voldev-expfunction.png)

The greater the absorption coefficient or the distance, the smaller T. The Beer-Lambert law equation returns a number in the range 0-1. If the distance or the absorption coefficient is 0, the equation returns 1. For very large numbers of either the distance or the density, T gets closer to 0. For a fixed distance, T decreases as we increase the absorption coefficient. For a fixed absorption coefficient, T decreases as we increase the distance. The further light travels in the volume, the more it gets absorbed. The more particles in the volume, the more light gets absorbed. Simple. You can see this effect in Figure 1.

<details>
<summary>Beer & gemstones.</summary>
An absorbing-only medium is transparent (not translucid) but dims images seen through it (e.g.: beer, wine, gemstones, tinted glass).
</details>

## Rendering a Volume Over a Uniform Background

It's easy to start from here. Imagine we have a slab of volume whose thickness and density are known. Say 10 and 0.1 respectively. Then if the background color (light reflected by a wall for example that we are looking at) is (xr, xg, xb), how much of that background color we see through the volume is:

```
vec3 background_color {xr, xg, xb};
float sigma_a = 0.1; // absorption coefficient
float distance = 10;
float T = exp(-distance * sigma_a);
vec3 background_color_through_volume = T * background_color;
```

It can not be simpler.

## Scattering

Note that until now we have assumed that our volume is black. In other words, we just darken the background color wherever our slab is. But the volume doesn't have to be. Volume like solid objects reflects (or scatters more precisely) light too. That's why, when you look at a cloud on a sunny day, you can see the shape of the cloud almost as if it was a solid object. Volumes can also emit light (think of a candle flame) which we are just mentioning for the sake of completeness but we will ignore emission in this chapter. So let's assume our slab of volume has a certain color say (yr, yg, yb). We will ignore where that color comes from for now; we will explain it later in the chapter. Let's just say until then that our volume has some color as a result of the volumetric object "reflecting" light (not really but let's go with the concept of "reflection" like with solid objects for now) that's illuminating it like with solid objects. Then our code becomes:

```
vec3 background_color {xr, xg, xb}; 
float sigma_a= 0.1; 
float distance = 10; 
float T = exp(-distance * sigma_a); 
vec3 volume_color {yr, yg, yb}; 
vec3 background_color_through_volume = T * background_color + (1 - T) * volume_color; 
```

Think of it as the process of blending (A+B) images in Photoshop for example using alpha blending. Say you want to add image B over A, where A is the background image (our blue wall) and B is the image of a red disk with a transparency channel. The formula to combine these two images would be:

$$
C = (1 - B.transparency) * A + B.transparency * B
$$

Where Transparency here is the 1 - Transmission (also called opacity) and B is the color of our volume object (light that's being "reflected" by the volume and traveling towards our eyes/camera). We will get back to this when we get to the ray-marching algorithm; for now, keep this in mind.

## Rendering our First Volume Sphere

We have all we need to render our first 3D image. We will render a sphere that we assume is filled with some particles using what we have learned so far. We will assume that we are rendering our sphere over some background. The principle is very simple. We first check for an intersection between our camera ray and the sphere. If there's no intersection, then we simply return the background color. If there is an intersection, we then calculate the points on the surface of the sphere where the ray enters and leaves the sphere. From there, we can compute the distance that the ray travels through the sphere and apply Beer's law to compute how much of the light is being transmitted through the sphere. We will assume that light "reflected" (scattered) by the sphere is uniform for now. We will look at lighting later.

![Figure 2: a camera ray passing through a volumetric object.](/images/volume-rendering-developers/voldev-simplesetup.png)

![Figure 3: we use the intersections points of the camera rays with the volumetric object to compute the opacity of the volumetric object along the camera rays.](/images/volume-rendering-developers/voldev-lightpassingthrough.png)

<details>
<summary>Implementation detail.</summary>
Technically, we don't need to compute the points where the ray enters and leaves the sphere to get the distance between the points. We simply need to subtract `tmin` from `tmax` (the ray parametric distances along the camera ray where the ray intersects the sphere. In the following example, we calculate them to emphasize that what we care about here is the distance between these two points.
</details>

```
class Sphere : public Object 
{ 
public: 
    bool intersect(vec3, vec3, float, float) const { /* compute ray-sphere intersection */ } 
    float sigma_a{ 0.1 }; 
    vec3 scatter{ 0.8, 0.1, 0.5 }; 
    vec3 center{ 0, 0, -4 }; 
    float radius{ 1 }; 
}; 
 
void traceScene(vec3 ray_origin, vec3d ray_direction, const Sphere *sphere) 
{ 
    float t0, t1; 
    vec3 background_color { 0.572, 0.772, 0.921 }; 
    if (sphere->intersect(rayOrigin, rayDirection, t0, t1)) { 
        vec3 p1 = ray_origin + ray_direction * t0; 
        vec3 p2 = ray_origin + ray_direction * t1; 
        float distance = (p2 - p1).length();  // though you could simply do t1 - t0 
        float tranmission = exp(-distance * sphere->sigma_a); 
        return background_color * transmission + sphere->scatter * (1 - transmission); 
    } 
    else 
       return background_color; 
} 
 
void renderImage() 
{ 
    Sphere *sphere = new Sphere; 
    for (each row in the image) 
        for (each column in the image) 
            vec3 ray_dir = computeRay(col, row); 
            pixel_color = traceScene(ray_orig, ray_dir, sphere); 
            image_buffer[...] = pixel_color;  // store pixel color in image buffer 
 
    saveImage(image_buffer); 
    ... 
} 
```

Quite logically, as the density increases, the transmission gets closer to 0 which means that the color of the volumetric sphere dominates over that of the background.

![](/images/volume-rendering-developers/voldev-simplevolspheres.png?)

You can see in the images above, that the volume gets more opaque towards the center of the sphere (where the distance traveled by the ray through the sphere is the greatest. You can also see that as the density increases (as sigma_a increases), the sphere becomes more opaque overall. Eureka! You've just rendered your first volumetric sphere and you are halfway to becoming a volume rendering expert.

## Let's add light! In-Scattering

So far we have a nice image of a volumetric sphere, but what about lighting? If we shine a light onto a volumetric object, we can see that parts of the volume that are more directly exposed to light are brighter than those that are in shadows. Volumes too are illuminated by lights. How do we account for that?

The principle is very simple. Let's imagine the fate of light emitted by a light source traveling through the volume. As it travels through the volume, its intensity gets attenuated due to absorption. And not surprisingly enough, how much is left of the light energy after it has traveled a certain distance in the volume is ruled by Beer's law. In other words, if we know the distance light traveled through the volume, its intensity at that distance is:

```
float light_intensity = 10; // just a number, it could be anything 
float T = exp(-distance_travelled_by_light * volume->absorption_coefficient); 
light_intensity_attenuation = T * ligth_intensity;
```

So first thing first, light energy decreases as it travels through the volume according to Beer's law. That's pretty logical. But something else also happens: light emitted by a light source that is not initially traveling toward the eye, can also be redirected toward the eye (at least part of it as we will see) as a result of what we call the scattering effect. We speak in this particular case of **in-scattering**. In-scattering refers to light passing through a volume, that's being redirected toward the eyes due to a scattering event. This effect is illustrated in Figure 4. A scattering event is the result of an interaction between a photon and a particle/atom making up the medium/volume. Rather than being absorbed or reflected (which can happen too), the atom just "spits out" the photon in a direction that's different from its incoming direction. We will learn more about this phenomenon in the next chapters.

![Figure 4: light we are seeing through volumetric objects comes from the background objects (here the blue color) as well as from light sources. Even though light beams when emitted by the light source are not traveling to the eye, some quantity of light is being redirected to the eye as it passes through the volumetric object due to the in-scattering effect.](/images/volume-rendering-developers/voldev-inscattering.png)

If you look at Figure 4, note that light that arrives at the eye (along the particular eye/camera ray that is drawn in blue in the figure), is a combination of light coming from the background (our blue background) and light coming from the light source scattered towards the eye due to in-scattering (the yellow ray).

So how do we account for the contribution of a light source? We need to "measure" light that's being scattered towards the eye (along with the camera rays) as the effect of in-scattering. The problem is that we need to account for that effect along the entire part of the camera rays that are intersecting the sphere (Figure 5). We need to **"integrate"** light that's being in-scattered along the camera ray, over the range t0-t1.

![Figure 5: we need to integrate the light that's be redirected towards the eye due to in-scattering along the segment of the ray that passes through the volumetric object.](/images/volume-rendering-developers/voldev-raymarching1.png)

To solve this problem, we will divide the section of the camera ray that's passing through the volume into a certain number of segments (our samples if you wish) and compute how much light arrives at the center of each one of these segments (sample) using the following procedure (see Figure 6 for a visual representation of the concept):

- We shoot a ray from that sample point (let's call it X) toward the light source to compute the distance from the sample point to the sphere boundary (let's call this point P). Note that X is always inside the sphere (our volume) and P is always a point on the surface of the sphere.

- Then apply Beer's law to know by how much the light energy was attenuated as it traveled from P (the point where this light ray entered the sphere) to X (the point along the eye ray where this light ray was scattered towards the viewer).

![Figure 6: marching along the ray in regular steps to estimate our integral using a Riemann sum.](/images/volume-rendering-developers/voldev-lightintegral1.png)

![Figure 7: we can estimate the area under the curve that represents the amount of light scattered along the camera using a Riemann sum. The idea is to break the area under the curve into a sum of small rectangles. The height of each rectangle is given by Li(x) and the width, dx, is user defined.](/images/volume-rendering-developers/voldev-lightintegral2.png)

To understand the type of problem we are solving here, we need to look at Figures 6 and 7. Figure 6 shows the incoming light arriving along the camera ray which, as you can see in the bottom part of the figure, is a continuous function. Let's call this function Li(x) where x here is any point along the camera ray contained within the range t0-t1. What we need to compute is the "area" below the curve. In mathematics, it is an integral that we can write as:

$$
\int_{x=t_0}^{t_1} Li(x) dx
$$

As we just said, the result of an integral (which is a number) is defined to be the (net signed) area under the curve (the function Li(x)) as illustrated in Figure 6. The problem in our case is that we can't compute this area using an analytical solution. But we can use a trick to approximate this area by breaking it down into simpler shapes we know the area of rectangles (as shown in Figure 7). We sample Li(x) along the curve at regular intervals which we know the width of (dx) and the area of the resulting rectangle can then be computed as Li(x) multiplied by dx (x is in the middle of the interval). By summing up the area of all the rectangles, we get an approximation of the area under the curve. Et voila! This technique is known as the Riemann sum (the idea of approximating a shape whose area we don't know with areas we do know goes back to the Greeks).

<details>
<summary>Going further.</summary>
You can find more information on integrals and how to compute them in the lesson ["Mathematics of Shading"](/lessons/mathematics-physics-for-computer-graphics/mathematics-of-shading/).
</details>

So how does it translate into code?
