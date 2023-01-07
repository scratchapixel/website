In this lesson, you will first learn what shading is and what are the different parameters that influence the appearance of objects, such as the light intensity or direction, the orientation of the surface of objects with regard to light sources, the color of objects, etc. Once these basic concepts are assimilated, we will be able to study a couple of very basic techniques to shade objects with and without light sources. We will then move to studying more complex shading techniques such as reflection and refraction. Finally, we will lean how to make the surface of objects visually more appealing using procedural texturing.  

This lesson is only a basic introduction to shading. In this section which is just an introduction to rendering, we won't scare you with too much of the theory involved behind the science of shading. We will keep it really simple and informal.

## Introduction to Shading

As mentioned many times in the previous lessons, the rendering process can be decomposed into essentially two steps: visibility and shading. We presented two techniques in the previous lesson to solve the visibility problem: rasterization and ray-tracing. From this point on, we will cover the second part of the rendering process: shading. Shading is a very large topics. Some of the techniques that are used in shading can also be mathematically complex. Though in this lesson, we will only cover the basics of shading and provide you with an introduction to the simplest and most important or common shading methods. This will provide you with an overview of what shading is and the necessary background to explore the topic deeper.

## What is Shading?

![Figure 1: computing the colors of objects in the scene is called shading](/images/shading-intro/shad-shading-principle.png?)

![Figure 2: at a first glance these pictures you see here looks like a professional photos, but if you look closer to some of them you?ll notice they are actually paintings. Talented artist Simon Hennessey has created such incredibly realistic paintings that looks more like photographs than paintings.](/images/shading-intro/shad-photorealistic-paintings.jpg?)

What is shading anyway? Shading is the process in rendering that consists of computing the color of objects in the 3D scene. In rendering we are interested in reproducing the shape, the visibility and the appearance of objects as seen from a given viewpoint. The visibility part of the rendering process deals with shape and the visibility problem. Shading deals with computing or simulating the color of objects as seen from a given viewpoint. The viewpoint here plays an important role. We will see in this chapter, that the appearance of objects can change with the angle of view. The appearance of an object also depends on some pretty obvious parameters such as the amount of light in the scene, the orientation of the surfaces with regard to the light sources in the scene, the objects' colors, etc. What contributes to defining the look or the appearance of objects will be reviewed in this chapter. Generally, why things look the way they do, has very much to do with the way light interacts with matter. The principles involved in this process are pretty well known today and are well described by laws which are also generally easy to simulate with a computer. Shading relies on these laws to simulate the appearance of objects in a consistent and realistic manner.

> In computer graphics, a shading function is defined as a function which yields the intensity value of each point on the body of an object from the characteristics of the light source, the object, and the position of the observer. Phong - 1975.

The level of **photorealism** of a painting depends on the painter's ability to reproduce the color of real surfaces on the canvas as seen by the human eye. If photo-realism is the goal, shading is concerned with reproducing the appearance or the look of real surfaces to such a degree of accuracy that the human eye can't make the difference anymore between looking at an image of the real world and the real world itself. The goal of **Photorealistic rendering** is to produce computer-generated image that looks real, or looks like a photograph. Shading which is the part of the rendering process during which the appearance of objects is defined, plays of course a critical role in photorealistic rendering. On the other side of the spectrum, **non-photorealistic rendering** (NPR in short) is concerned with creating or simulating all sorts of artistic rendering styles. The goal of NPR can be to reproduce real artistic techniques such as oil or watercolour painting, or to develop entirely new styles that can only be explored and generated with computers. In the lessons from this section, we will only be interested in photorealistic rendering.

<details>
The line between shading and rendering as in "the action of simulating with a computer the creation of images produced by real-world cameras" is often very thin. For example, in the following lessons, we will study how to simulate the effect of motion blur and depth of field, which strictly speaking, have nothing to do with shading. Though they don't have much to do with solving the visibility problem either. These topics often lie in a grey zone that sort of relates to rendering as a whole. So the term rendering for the lack of a better word can also be interpreted as anything that doesn't relate to solving the visibility problem. This would include shading and effects such as depth of field, lens flare, motion blur, etc.
</details>

## Setting Up Terminology

![Figure 3: light is reflected by objects. The eye sees this reflected light. This is an example of direct illumination.](/images/shading-intro/shad-reflected-light1.png?)

![Figure 4: light reflected by objects can illuminate other objects in the scene. This is called indirect illumination.](/images/shading-intro/shad-reflected-light2.png?)

![Figure 5: the number of objects light may interact with before it reaches the eye can in theory be infinite and/or in practice very large.](/images/shading-intro/shad-reflected-light3.png?)

When the image is generated by a computer, then we can't rely on the painter's eyes to decide about the color each pixel in the image should be set to. Hopefully, the appearance of objects, which is what we want to simulate in photorealistic rendering, is "just" the by-product of essentially two things: lighting and the properties of the object, and these two things as well as their interaction can be simulated with a computer. You can't see objects if there is no light, so light plays an essential part in the process. And the more light, the brighter the objects. As for the object's properties, we can essentially decompose them into two main categories: the geometric properties of objects' surfaces (such as their orientation) and the properties that are involved in the way light and objects interact (such as, the object's color).

When we "see" an object, what we see is light reflected off of the surface of that object. Most objects don't emit light. Generally, light comes from what we call light sources, special objects which have the property of emitting light (the sun, light bulbs, and the flame of a candle are all examples of light sources). Typically, light is emitted by light sources, falls on the surface of an object (that doesn't emit light), and this object reflects some of that light in the direction of the viewer (figure 1). This light forms in our brain the image of the object we look at. In shading, this is what we call **direct lighting**, or **direct illumination**. Though since objects reflect light that falls onto them, it doesn't matter if this light comes directly from a light source or indirectly from other objects in the scene. When an object reflects light towards the surface of another object, which in turn reflects this light toward the viewer (or at least a fraction of it as we will soon explain), we speak of **indirect illumination** (figure 4). Note that light can bounce off of many surfaces before it gets finally reflected toward the viewer (figure 5).

## Light-Matter Interaction and the Essential Components of Shading: N, P, L, and V

![Figure 6: the brightness of a point on the surface of an object depends on the orientation of the surface normal at this point with regard to the incoming light direction. Intensity reaches a maximum peak of intensity when the light direction and the surface normal are parallel.](/images/shading-intro/shad-light-surf-orient.png?)

This quick introduction gives us already some clues about what's happening in the process. What we see are not objects _per se_, but light reflected off by the surface of objects. Light is energy and is emitted by light sources and reflected off the surface of objects. Now, as mentioned before, the orientation of surfaces plays an important role in the amount of light reflected off. We all know that if we turn a surface towards a light source it will become brighter or if we turn it away from this light source it will become darker. Surface orientation (about light sources), as we will see, plays an important role in shading. This is where the normal \(N\) at the surface of an object is going to be of most use to us. Of course, we can change the orientation of a surface in relation to light or change the position of the light source in relation to the object's surface. Changing the surface orientation or the light position (and thus its direction with respect to the surface orientation) has the same effect. The light direction \(L\) can simply be defined by tracing a line (a vector) from the point \(P\) on the surface we want to compute the color of, and the light source position (figure 7). For now, we will assume that lights are infinitesimally small points in 3D space. We don't know yet how to compute the color at \(P\), but we know it depends on \(P\), the point on the surface we want to compute the color of, \(N\), the surface orientation at \(P\) which is given by the surface normal \(N\) at \(P\), and \(L\) the light direction (of course how "bright" is the light source also plays a role in how bright \(P\) is but we will talk about this later). What else is important?

![Figure 7: the angle of incidence and the angle of reflection are equal if the surface is a mirror.](/images/shading-intro/shad-mirror.png?)

So far, we explained that light was reflected off of the surface of objects. How is light reflected is something we will study soon, but what about understanding in which direction light is reflected to? The direction in which light strikes the surface is called the **incoming** or **incident light direction**. The angle this vector forms with the normal \(N\) at \(P\) is often denoted \(\omega_i\) (the Greek letter omega and the subscript \(i\) stands for "incoming" - the Greek letter \(\theta\) is also sometimes used but we will stick with the convention of using \(\omega\) which other educational resources such as PBRT use as well) and is often called the **angle of incidence**. Perfectly reflective surfaces, e.g. mirror-like surfaces, reflects light exactly like a tennis ball bouncing off of the ground of a tennis court. The outgoing direction of the ray forms an angle with the normal \(N\) at point \(P\) which is generally denoted \(\omega_o\) (where \(o\) in this case stands for "outgoing"). This angle is also called the **angle of reflection**. The angle of reflection and angle of incidence for the mirror-like surface is governed by the **law of reflection**. It states that **when a ray of light reflects off a surface, the angle of incidence is equal to the angle of reflection** (figure 7). Almost everything in shading is somehow derived from this simple law (though this doesn't mean things will be that simple later on).

If the surface is a perfect mirror, then we can easily find the outgoing light, mirror of reflection direction \(R\) (depending on how you like to call it) using some basic geometry. The complete equation to compute this reflection direction will be provided in the next chapters.

Let's define the viewing direction \(V\), as the line joining \(P\) and the eye (\(E\)) or camera origin as shown in figure 6\. Note something interesting though which is that if a viewer doesn't look exactly at \(P\) in the same direction as the outgoing light direction (i.e. if R != V), then the light ray won't enter the viewer's eye. In other words, the viewer "won't see the light ray or the image of the light source emitting that light ray". It will miss the viewer (figure 7). This is an important observation and we will come back to it in a short while (keep it in mind for now).

![](/images/shading-intro/shad-materials.png?)

![Figure 8: glossy surfaces are "broken" mirrors. Their surface is not perfectly smooth and can be seen as a collection of small facets all acting individually as small mirrors but more or less oriented in random directions](/images/shading-intro/shad-rough.png?)

![Figure 9: different levels of glossiness or roughness.](/images/shading-intro/shad-roughness1.gif?)

![Figure 10: the slightly rippled surface of a lake acts as a rough surface. The image of the reflected background is slightly blurred.](/images/shading-intro/shad-roughness2.png?)

Few surfaces in the real world is perfect mirrors though. Most surfaces are somehow **shiny** or **glossy** (depending on the term you prefer to use) or matte or **diffuse**. Again here the terms can be used interchangeably though, in CG, we prefer the term **diffuse**, or **Lambertian** when it comes to matte surfaces. Both terms will be explained soon. What makes a surface glossy rather than mirror-like or diffuse? Glossy surfaces are "broken" mirrors if you wish. Their surface is not perfectly smooth and can be seen as a collection of small facets all acting individually as small mirrors but more or less oriented in random directions as shown in figure 8. As a result, rather than being reflected exactly in the mirror direction, light rays are reflected in directions slightly different than the ideal mirror direction. How far off from the mirror direction they are reflected off depends on how different the facet's orientations are from the "mirror-like" surface plane. The greater the difference, the more likely reflected rays will diverge away from the mirror-like direction (and the glossier the surface).

The effect of the rays being reflected in other directions than the mirror direction is to blur the reflected image. Or to say the same thing but the other way around, a blurred reflection on the surface of an object is the result of the surface being broken up or rough and reflecting light in over directions than the mirror direction. The rougher the surface, the blurrier the reflected image (Figures 9 and 10). In summary, materials can go from a perfect mirror surface to a very rough surface as shown in the animation of figure 9.

![Figure 11: a diffuse material.](/images/shading-intro/shad-diffuse1.png?)

![Figure 12: for diffuse materials, light is reflected equally in all directions above the surface.](/images/shading-intro/shad-light-beam5.png?)

Diffuse materials can be seen as the exact opposite of perfect mirrors, and you may be tempted to think that diffuse materials are very rough materials. Though why they are diffuse has generally little to do with roughness itself. Diffuse materials are often materials that exhibit complex internal structures. Because of this, light rays get "trapped" for a while in the object's matter, bouncing back many times against the surfaces of that structure before eventually leaving the object. Light rays bounce so many times against these internal structures that their direction when they leave the surface is uncorrelated with their incident direction. In CG, we like to say that their direction when they leave the surface of the object at \(P\) is random. On 100 rays hitting the surface in a very small area around \(P\) with the same incident direction, we end up with generally less than 100 rays leaving the surface in random directions (less because some of the rays may have been absorbed by the object but this is another topic that we will cover soon).

For this particular reason, we like to see diffuse objects as reflecting incident light equally in every possible in hemisphere of directions centered around \(P\) and oriented about the normal \(N\) at \(P\) as shown in figure 12\. This also means that there is a fundamental difference in the way glossy and diffuse materials react to light. Because diffuse materials reflect light equally in all directions, their brightness doesn't change with the view direction, which is not the case with glossy materials. Remember what we said before about the view direction and the mirror reflection. If they are not the same the reflected ray doesn't enter the eye and we don't see the image of the source from which the light ray was emitted. Similarly, the image reflected by a mirror will change as you change your point of view with respect to the mirror. Reflection and glossy reflections (which are just reflections by rough surfaces) are said to be **view-dependent**. What you see of an object or light source is reflected by a glossy surface depending on your angle of view. Diffuse reflections (it is also a sort of reflection in the sense that light is reflected by the object in the environment) on the other hand are **view independent**. You can look at them from every angle you want, their brightness won't change (and this is because diffuse material reflects incident light equally in all directions). This distinction is very important. You can experience it easily by looking at some glossy and diffused objects around you. Notice how their brightness changes or not depending on your viewpoint.

![](/images/shading-intro/shad-view-dependent.png?)

![Figure 13: the skin of an orange absorbs most of the blue colors, leaving only red and green light which mixed together (in additive mode) form an orange color.](/images/shading-intro/shad-orange.png?)

The last component of shading we need to quickly cover is to explain why objects have color and how we can simulate it. An entire lesson is devoted to the topic of color, color perception, etc. We will only quickly review the process in this lesson. We will assume here that you are familiar with the idea that "white" light is composed of all the spectral colors composing the visible light spectrum. When "white" light strikes an object, some of these colors are absorbed by the object and the others are of course reflected in the environment. The spectral colors that are absorbed by the object are a property of the object material itself. An orange fruit for example is orange, because it absorbs most of the blue colors, leaving only red and green light which are mixed (in additive mode) to form an orange color (it absorbs a bit of the green colors as well which is why the fruit is more orange than yellow). As you can see, a given material can absorb all or just a fraction of some colors. The orange fruit absorbs almost all the blue colors (say 90%), absorbs a fraction of the green colors (say 40%), and only a very small amount of the red colors (10%). The light that is not absorbed is reflected (10% of blue plus 60% of green plus 90% red). But how can we define the color of an object? First, note that the concept of color only makes sense if the object is somehow diffuse. Though to make things complicated, most materials in the real world exhibit both behaviors to some extent: they have a diffuse component and a glossy component. They are rarely one or the other completely. So mirrors don't have a color _per se_. Orange fruits are more complex. They have a color but they look shiny as well. This is because the orange skin which is diffuse for the most part is covered by a very thin oily layer which is acting as a mirror. The oily, transparent layer reflects some of the light falling on the fruit, but some light is also transmitted through the oily layer to the diffuse skin layer which reflects light as a diffuse surface. But the color of an object can be defined as the ratio of reflected light over the amount of incident light. In the orange fruit example, this ratio would be 0.1 for blue, 0.6 for green, and 0.9 for red (assuming an RGB color system to represent colors). The technically correct term to speak of the color of an object is **[albedo](http://en.wikipedia.org/wiki/Albedo)**.

## Summary

In this chapter, we quickly reviewed the very basic principles of light-matter interactions and laid down the terminology we will be using in shading. These principles of light-matter interactions are important because it is essentially what we will need to simulate to produce photo-realistic images. You should remember that light is reflected by objects. On a perfectly flat mirror-like surface, the reflection direction is given by the law of reflection.

## Terminology

|-table{Name,Quick Description}
|-row
|-cell
P
|-cell
The point that's being shaded.
|-row
|-cell
N
|-cell
The normal at P. We often use the letters T and B to indicate the tangent and bitangent at P.
|-row
|-cell
L
|-cell
Light direction.
|-row
|-cell
V
|-cell
View direction.
|-row
|-cell
Angle of Incidence
|-cell
The angle between N and L.
|-row
|-cell
The angle of Reflection.
|-cell
The angle between the reflected ray and the normal.
|-