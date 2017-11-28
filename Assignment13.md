# Assignment 13
### A Small One

In this assignment we had to convert our asset build pipeline to something more efficient.  Previously, the build pipeline would rebuild all assets every time the "BuildExampleGameAssets" project was executed.  The goal of this assignment was to replace that with a smarter approach.

## Purpose

The previous asset build pipeline was inefficient.  It would rebuild all assets even if they hadn't changed!  This proved to be time-consuming.. and the ShaderBuilder was currently carrying the bulk of the processing time.  Building the shaders in some cases, for about 5 shaders, would take around 3-7 seconds!  The obvious solution to the problem, of course, is to only build the assets that need to be built.

## The Assignment

We were given some code to integrate into the asset building projects and with the usual "TODOs" to fill in scattered about.  One file of interest is the game-specific "AssetsToBuild.lua" file.  Here's the one I used for my ExampleGame project:

``` lua
--[[
	This file lists every asset that must be built by the AssetBuildSystem
]]

return
{
	meshes =
	{
		"Meshes/Floor Plane.lua",
		"Meshes/Rainbow Cube.lua",
		"Meshes/Kamikaze Enemy.lua",
	},
	shaders =
	{
		{ path = "Shaders/Vertex/sprite.ppsl", arguments = { "vertex" } },
		{ path = "Shaders/Fragment/sprite.ppsl", arguments = { "fragment" } },
		{ path = "Shaders/Vertex/mesh.ppsl", arguments = { "vertex" } },
		{ path = "Shaders/Fragment/mesh-vertex-colored.ppsl", arguments = { "fragment" } },
		{ path = "Shaders/Fragment/mesh-textured.ppsl", arguments = { "fragment" } },
	},
	shaders_d3d =
	{
		{ path = "Shaders/Vertex/vertexInputLayout_mesh.ppsl", arguments = { "vertex" } },
		{ path = "Shaders/Vertex/vertexInputLayout_sprite.ppsl", arguments = { "vertex" } },
	},
	textures =
	{
		"Textures/walk1.png",
		"Textures/bricks.png",
		"Textures/kamikaze.png",
	},
}
```

This has many benefits.  The first, and most important, is that only these assets are built for the ExampleGame project, and they're **only** built if they are out-of-date (because you edited something).  If you examine the previous uploads you will notice I was still building and delivering the "walk" textures despite only using one of them for the last six or so assignments.  (This was mainly because I didn't remove the build entry from the former asset system.)  Now, you should only see the assets listed above in the release build.

Also, while it wasn't a part of the assignment, I added platform-specific assets.  It should be obvious, above, to see that the `shaders_d3d` stands out from the other listed assets.  The assets within are **only** built for the Direct3D platform since they are only needed there (and not on the OpenGL platform).  Currently, one can declare any "platform-specific" assets above by following that pattern: `[asset type]_[platform]`.  Right now, there's only two "platforms":  d3d and gl.

All-in-all, this new system prevents the unnecessary "rebuilding" of assets that haven't changed since last build.  This "smart asset building" is definitely a requirement on larger projects when the amount of assets can number in the hundreds or even thousands.

## Wrapping Up

Not much has changed from the user's perspective, however.  Still, here's a screenshot demonstrating a bit of what can be done "in game".

![Assignment 13 gif](images/a13/assignment13.gif)

[Windows - Release - Direct3D](https://github.com/CorneliaXaos/EAE6320-WriteUps/releases/download/a13/Assignment13.zip)

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
* Reset the Camera
  * Press **R**!
  
Enjoy!
