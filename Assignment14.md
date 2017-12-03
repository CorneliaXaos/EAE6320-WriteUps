# Assignment 14
### Now I Need to Find a Ghost Mesh...

In this assignment we had to add translucent (and, consequently, transparent) mesh support to our engines.  This would allow for having objects in our game scene that don't fully obscure the other objects behind them.

## Purpose

The purpose of this assignment was really to just add an interesting feature to the engine.  A lot of interesting geometry can exist in a game that needs some form of translucency.  An example could be a waterfall or stained glass windows...  Though our implementation doesn't account for lighting.  (In fact, everything in the engine is currently rendered in "fullbright" mode..  There is no lighting calculations being done.)

## An Early Gif

So here's the screencap for this assignment.  I'm showing it early as I shall be referencing it in the next section:

![Assignment 14 gif](images/a14/assignment14.gif)

## How It's Done

The following is a description of how I added translucency to the engine;  there are quite a few ways translucent / transparent objects can be rendered, but the method I implemented focused on ordering the translucent *meshes*.

### Update Fragment Shaders?

The Fragment shaders that will render the translucent meshes need to be capable of outputting a color value that indicates translucency.  In my case, my two mesh shaders, `mesh-vertex-lit` and `mesh-textured` already output the entire four-float color value (from either the vertex color itself or the texture sample).  The fourth value is the alpha value for that fragment and is used to perform alpha blending.

### Effect Render State

In order for alpha blending to occur, the `Effect` objects we create in our engine have to be told that Alpha Blending of translucent colors is desired.  We do this by enabling "Alpha Transparency" in their Render State.  This is made easy through the use of my `EffectBuilder` implementation which allows for setting the Render State bit flags easily.

### Meshes Are Submitted As Normal

From the gameplay programmer's perspective, nothing much has changed.  Sure, the `Effect` now has to be built with "Alpha Transparency" support enabled.. but meshes are still submitted as normal.

The difference is what happens when the mesh is submitted.  The Graphics project code that receives the incoming mesh checks the `Effect` that was submitted with it.  If the Render State flags indicate that "Alpha Transparency" is desired, that mesh is submitted into a different queue than the opaque geometry.  This is important as opaque geometry can be rendered before all the translucent geometry.

### Graphics Rendering

When the Graphics project goes to render it first renders all the opaque geometry.  This is done so that the depth buffer is populated with some useful values.  This makes it easier to cull unnecessary translucent fragments from the more structured rendering below.

Before the queue of translucent geometry can be rendered, however, it needs to be sorted.  We sort the geometry "from back to front".  What this means is that, in camera space, the objects at the "back" are furthest from the camera.  This is the order we want to render our translucent geometry as it allows for properly blending translucent geometry together (with some caveats mentioned below).

I sort the geometry on the render thread using a fancy compare closure.  It produces a `std::function` that takes two `MeshJob` pointers as arguments and returns true if the first is further from the camera than the second along the Z-axis.  It does this by converting the two `MeshJob` transforms to camera space and comparing the z-values of their translations.

After the translucent geometry is sorted, it is rendered in its sorted order.  Again, this allows for translucent geometry to be blended together appropriately.

Finally, the sprite / screen-space geometry is rendered last.

## Comments on This Approach

Translucent geometry is a bit of a headache for designers, developers, and graphics programmers.  Correctly rendering scenes in a way that mimics reality (i.e. lighting) is already a difficult task.  Translucent geometry has to be treated differently in most cases to ensure that the result that is produced is at least believable by the user.

The algorithm described above is very similar to the ["Painter's Algorithm"](https://en.wikipedia.org/wiki/Painter%27s_algorithm).  In fact, the ordering of the translucent geometry, rendering it in that order, and finally rendering the screen-space "sprite geometry" is just that.  It's a very simple algorithm to implement.  This makes it very easy to put into code in the timespan of an hour or two... but has some drawbacks..  Notably, that it can't handle certain types of complex geometry.

In the gif above, about halfway through, I stop and move the camera back and forth for about five seconds.  The intersecting translucent objects seem to flicker about their intersection.  This is because the algorithm, as I implemented it, cannot handle intersecting geometry well.  This is because all of the triangles of the mesh are considered in front of or behind another mesh's triangles based solely upon the pivot of those two meshes.  Basically, we are treating the meshes as point objects.  This can lead to some interesting behaviour for intersections, and even more unusual geometry like horseshoes or toruses with something going through the center.  One solution to this is to avoid intersecting translucent geometry.

There are alternative solutions, however.  One solution that is somewhat similar to this one is to sort all of the **triangles** or **fragments** instead of the **meshes** themselves.  An even more advanced solution (though less likely to be used in games because of its time complexity) is Depth Peeling.  It is worth reading up on, but I won't explain it here.  There are [some](https://en.wikipedia.org/wiki/Depth_peeling) [great](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.18.9286&rep=rep1&type=pdf) [resources](http://martindevans.me/game-development/2015/10/09/Deferred-Transparency/) [available](http://www.opengl-tutorial.org/intermediate-tutorials/tutorial-10-transparency/) elsewhere, though.

## Conclusion

And that's all folks!  Here's the download link for this build and a recap of the controls!

[Windows - Release - Direct3D](https://github.com/CorneliaXaos/EAE6320-WriteUps/releases/download/a14/Assignment14.zip)

### Controls

You can use the following controls to:

* Translate The Camera
  * **W** and **S** will move the camera forward and backward.
  * **A** and **D** will strafe the camera left and right.
  * **Q** and **E** will strafe the camera down and up.
* Rotate The Camera
  * **I** and **K** will rotate the camera about the x-axis (pitch).
  * **J** and **L** will rotate the camera about the y-axis (yaw).
  * **U** and **O** will rotate the camera about the z-axis (roll).
* Translate the Brain-Creature-Thing...
  * The **Up** and **Down** arrow keys will translate along the World Space y-axis.
  * The **Left** and **Right** arrow keys will translate along the World Space x-axis.
  * The **Page Up** and **Page Down** keys will translate along the World Space z-axis.
* Reset Everything
  * Press **R**!
  
Enjoy!
