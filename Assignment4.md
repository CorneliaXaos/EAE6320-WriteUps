# Assignment 4
### Making Things More "Engine"

The objectives of this assignment involved abstracting things just a little further.  We needed to make our `Sprite` and `Effect` implementations reference counted as well as make it so that the Graphics Project didn't own any `Sprite`s or `Effect`s itself; rather, whatever game project was using the Graphics project should create and own the `Sprite`s and `Effect`s and submit them to the application for rendering.

## Purpose

What I've mentioned in the previous write-ups has finally come to pass; the purpose of this assignment was to finally make the Graphics engine more "engine-like" and data-driven!  After completing this, you can use the Graphics project to render an arbitrary number of `Sprite`s with any kind of `Effect`s you can conceive!  (Obviously, there's still some limitations such as "texturing" of the `Sprite`s.)  Still, the ultimate goal of this assignment was the make the Graphics engine an actual engine that can be driven with data it does not own.

## Reference Counting

One of the big pillars of this assignment was to make the `Sprite` and `Effect` classes reference counted.  This was primarily because of the need to pass their references around between threads.  There are many ways this could have been done (smart pointers, etc.) but the limitations of the assignment stated that we needed to use a particular implementation of reference counting macros provided in the Assets project.  Still, I added a convenience method to that implementation to make things somewhat more easy for me.

In the end, my `Sprite` and `Effect` implementations had the following sizes:

| `sizeof()` | OpenGL | D3D |
| ---------- | ------ | --- |
| `Effect`   | 20     | 48  |
| `Sprite`   | 12     | 24  |

This was as small as I could make them on each platform.  The primary difference in sizes between OpenGL and D3D is due to the D3D impelmentation tending to require pointer members and being a 64bit build.  This usually means that each pointer tended to have a size of 8 and took up quite a bit of space.  I tried to minimize the sizes as best as I could by ordering the declaration of the member variables in decreasing size so that they could be packed together most efficiently.

## Data Submission

### Background / Clear Color

One of the requirements for this assignment was to have a way to instruct the Graphics renderer to clear with a specific color.  So I developed two: one method that could be called during initialization to set the render color for both data frames and one to set only the current one being submitted to.  They are as follows:

``` c++
eae6320::Graphics::SetClearColors(sColor); // sets the clear color in both frames, used for initialization
eae6320::Graphics::SubmitClearColor(sColor); // sets the clear color in the current "submit to " frame
```

My Example Game project makes the following call to set up the clear colors **once** during the initialization of the ExampleGame application instance:

``` c++
// Namespace stuffs up here
namespace Graphics = eae6320::Graphics;

...

eae6320::cResult eae6320::cExampleGame::Initialize()
{
	auto result = eae6320::Results::Success;

	Graphics::SetClearColors(Graphics::sColor{1.0F, 0.5F, 0.0F});

// Rest of cExampleGame.cpp
```

### Data Frames?

To clarify, the Graphics project contains two "data frames" or buffers into which data for rendering can be submitted.  One of those frames is being rendered from while the other is being submitted to by the main application.  This is required for efficiency purposes as the renderer essentially has to coordinate two "computers" running in parallel: the main CPU processing the application where the renderer itself lives, and the GPU which is processing the graphics data and creating pretty pictures for us to look at.. :P

If this technique was not used, then the efficiency of the application would be extremely impacted.  Before new data could be submitted to the renderer all the old data would have to be cleaned out and sent to the GPU for rendering.  By having two buffers we can manipulate, we can write to one while the other is being rendered.  This alleviates the inefficiency that comes with talking to the GPU by hiding it in its own thread behind some shared data pipelines.

### Submitting Sprites / Effects

Finally, the Example Game project needed to own and submit the `Sprite`s and `Effect`s to the Graphics renderer in place of the renderer owning them and having them hard-coded into the engine.  This was the final piece of the puzzle required to make something that could be multi-purpose and used for many different games (albeit, right now, they'd all be games involving colored rectangles as that's the best you could do with what we've done).

In the end, the code I use to submit the `Sprite`s and `Effect`s for rendering is as follows:

``` c++
// back up near the namespace shortenings
namespace {
	// Effects
	eae6320::Graphics::Effect* renderWhite = nullptr;
	eae6320::Graphics::Effect* renderRainbow = nullptr;

	// Sprites
	eae6320::Graphics::Sprite* upperRight = nullptr;
	eae6320::Graphics::Sprite* lowerLeft = nullptr;
	eae6320::Graphics::Sprite* middle = nullptr;
	
	...
}

    ...

    // inside the submission function
	Graphics::SubmitSpriteBatch(renderWhite, {upperRight, lowerLeft}); // Render Batch! :D
	Graphics::SubmitSprite(renderRainbow, middle);

// Rest of cExampleGame.cpp
```

So.. I did a little more than the assignment asked; I created a method in the Graphics Project to submit a single `Effect` / `Sprite` pair.. but I also created a way to submit a batch!  The `SubmitSpriteBatch` takes a single `Effec`t and a list of `Sprite`s that are all then rendered (in the order provided!) using the same `Effect`!  Pretty neat if I do say so myself. :P

All in all, the user can call those submission methods as much as he or she wants and submit an arbitrary number of `Sprite`s and the `Effect`s with which to render.  The only limit is the memory allocated to the application!  This is accomplished by using a STL Queue behind the scenes to cache `RenderJob`s.  These are held on to until the Renderer has a chance to render them, or the application exits and the data frames are flushed.

## Wrapping Up

And that's what was done this assignment.  As usual, here's a gif of the application running as well as a link to the download for the binary itself.

![Assignment 4 Gif](images/a04/assignment4.gif)

[Windows - Release - Direct3D](https://github.com/CorneliaXaos/EAE6320-WriteUps/releases/download/a4/Assignment4.zip)

~ Cornelia Schultz
