Documentation Author: Niko Procopi 2019

This tutorial was designed for Visual Studio 2017 / 2019
If the solution does not compile, retarget the solution
to a different version of the Windows SDK. If you do not
have any version of the Windows SDK, it can be installed
from the Visual Studio Installer Tool

Welcome to the Geometry Ray Tracing Tutorial!
Prerequesites: 
	"More Graphics" section, 
	Instanced Render Particles (procedurally-generate a quad)

Quick tip: try opening GLSL files in Notepad++,
then go to Language -> C -> C, to color-code the shader

In this tutorial, we will show the basics of Ray Tracing,
we will draw geometry on the screen (one plane and two cubes),
without any lighting or texture effects. This tutorial
will explain the main.cpp file, the Vertex Shader file,
and the Fragment Shader file. For the next two tutorials
after this, only the Fragment Shader will change, so it 
is very important that main.cpp and the Vertex Shader are
understood in this tutorial.

[Structure of Main.cpp]
{
	Initialize GLFW,
	Make a window,
	set swap interval,
	
	call init()
		Load the shader files,
		compile the shaders,
		attatch them to a shader program
		link the program
		activate the program (glUseProgram)
		create uniform for "eye" and four rays
		call calcCameraRays (for four rays)
		set all 5 uniforms (eye and four rays)
	
	enter while loop
	
		call update()
			update elapsed time
			calculate frame rate
			change window title
			update camera position (eye)
			call calcCameraRays
			update all 5 uniforms
	
		call renderScene()
			clear screen
			draw 4 vertices (no VAO or VBO needed)
}

Just like deferred rendering, we use the Vertex Shader
to procedurally-generate geometry, without any buffers,
to cover the entire screen. After the screen is covered,
the rasterizer tells the pixel shader to activate for every
pixel on the screen.

The Fragment shader is given the camera position (eye), and
four different rays as. Each ray is one of four corners
of the screen, in the shape of a frustum (look up "frustum"
in Google Images).

In Ray Tracing, we do not have the model, view, or projection
matrices. Those need to be replaced. Just for a recap, lets
go over what information was in those matrices (we will need
to use it here).

Projection matrix:
FOV of camera
aspect ratio of screen
nearest distance to camera before clipping
farthest distance from camera before clipping

View Matrix:
glm::lookAt(eye, center, up)
eye: point where the camera is
center: point that the camera is looking attatch
up: the direction that faces up (rotation of camera)

When we call our calcCameraRays function, 
we pass FOV, aspect ratio, eye, center, and up
as parameters. These will act as input. We also
pass four pointers, which will be our output,
which are the four rays that make the view
frustum (one ray in each corner of the screen).
Nearest and farthest distance will be 
hard-coded later on

Camera position simply rotates around the world
the position we focus on is (0, 0.5, 0), which is
slightly above the origin of the world.
Our up vector is (0, 1, 0), which means the Y-axis
points upward. Our aspect ratio is 800/600, which
is window width divided by height, then we pass
pointers to our 4 vec4 rays (r00, r01, r10, r11).
After that, we set the Uniforms, which are explained
in prerequisites.

[Vertex Shader]
We generate a quad to cover the screen by using 
gl_VertexID; we put one vertex at each corner,
we pass position and UV for each vertex.

[Fragment Shader]

We take uniforms as input from C++,
we take "textureCoord" as input from Vertex Shader,
we output a "color" to the screen

We create a structure called "triangle", this is
not an ordinary triangle. Previously, we had 
Vertex Buffers, where each vertex had a position, 
a color, and a normal. This triangle has three 
vertices, where the vertex positions are vec3 a, 
vec3 b, vec3 c, while the triangle has one color, 
and one normal, that are shared by each of the three
points.

We set the maximum distance that our rays will check for,
as 100.0 units. Anything beyond that point will not be drawn.
Nothing in the scene will be farther than 100.0 units anyways,
and we set NUM_TRIANGLES to 26

Here comes the crazy part, we hard-code every triangle
in the Fragment Shader.
	const triangle triangles[] = triangle[26](...)
This array contains 26 triangles, 2 for the blue plane,
and 12 for each of the red cubes. The reason we did this
was to make a point, we want it to be absolutely clear that
polygons are processed in the fragment shader, not the 
vertex shader. Eventually, when we make an "Advanced Ray 
Tracing" section, we will pass triangles to the shader
through buffers that are made on the CPU, to load 3D models

We create a structure that gives us information on what
happens when a ray hits a triangle. This gives us the
position of the hit, and the index of the triangle array,
to tell us which triangle it hit
	struct hitinfo
	{
		vec3 point;
		int index;
	};
	
We have a function called rayIntersectsTriangle(...) which
checks one triangle for collision (more details later).

We have function called intersectTriangles(...), which
checks all triangles for a collision, by calling
rayIntersectsTriangle multiple times. 

We have a function called trace(...), which sends a ray
from a starting point, into the scene.

In main(), we take in the texture coord from the vertex shader.
Because the rasterizer interpolates this value, we now have
the screen position for each pixel (0 to 1).

The deferred rendering tutorial uses one triangle to fill
the screen in the vertex shader, then uses gl_FragCoord.xy,
and divides FragCoord by screen resolution to get the position
of each pixel, while this tutorial uses UV coordinates
that are interpolated from a vertex shader that interpolates
two triangles (one quad). Both methods give the same result,
it is only a matter of preference.

Now that we have the position of every screen pixel, we need
to use the interpolated position data to generate interpolated rays.
We use our 4 rays, and our interpolated position, to make one
ray for every pixel (vec3 dir). After that, each pixel
uses the "trace" function to send a ray from the eye, into the scene,
in the direction that we just calculated for this particular pixel.

In this simple example, we have one ray per pixel. Each ray will launch
into the world, and check for collision with all polygons. If a ray
collides with a triangle, then the color of this pixel will be set to
the color of the that triangle that the pixel's ray hit. "trace"
returns a color, and the "color" output of the fragment shader is
set to the color that we get from "trace".

trace(vec3 origin, vec3 dir):
We create an instance of hitInfo, and we pass it to the function
intersectTriangles. This checks for an intersection with every 
triangle in the scene, and returns "true" when a polygon is hit, 
or returns "false" if no triangle is hit. 

If intersectTriangles returns true, we take the "index" variable
of hitInfo, it tells us which triangle the ray hit, and then 
we get the color of that triangle: 
	triangles[i.index].color.rgb
If the ray does not hit anything, if intersectTriangles returns
"false", then we return black 
	vec4(0.0, 0.0, 0.0, 1.0);
If you want to change the background color, this is where
you would change it. You can also return vec4(dir, 1) for
a cool effect.

bool intersectTriangles(vec3 origin, vec3 dir, out hitinfo info)

We don't have a depth buffer in ray tracing, so we need to manually
find the polygon closet to the camera. We need to check each polygon 
for collision, and then out of all the polygons it hit, we check to 
see which polygon is closet to the cameara. In the future when we have
an "Advanced Ray Tracing" section, we will have optimizations so that
rays don't need to check every triangle, but only nearby triangles.

MAX_SCENE_BOUNDS is the farthest distance that we care about,
this is the equivalent to the farthest distance that we would put
in a projection matrix with glm::persepctive(fov, aspect, near, far)

First we create a float called "smallest" for the smallest distance
between the polygon and a camera (if we ever hit a polygon). We set 
this distance to MAX_SCENE_BOUNDS, assuming that a polygon with a distance
smaller than MAX_SCENE_BOUNDS will take its place. We create a bool "found",
to determine if any polygon is hit.

We loop through every triangle:
	for(int i = 0; i < NUM_TRIANGLES; i++)
	
We get a float from rayIntersectsTriangle, which checks to see if the ray
intersected with the current triangle we are looking at in the "for" loop.
we set "float t" equal to the value returned from rayIntersectsTriangle.
rayIntersectsTriangle returns -1 if it hits nothing. Therefore, if 't' is
not equal to -1, and if it is smaller than MAX_SCENE_BOUNDS (or smaller than
the closet polygon so far), then set 't' equal to "smallest". This triangle
is now the closet polygon we've collided with so far. We also set the point
of our collision (info.point) to "origin + (dir * t)", which is the position 
we started from, plus direction * distance, which gives us the point of intersection, 
we set the index of our collision (the index in the array of triangles to tell us
which triangle we hit) info.index = i, and we set "found" to true.
We continue this loop, and if we find anything with a closer distance, we keep
overwriting the values.

By the time the loop is done, we return "found" to tell "trace" if we hit anything.
If we did hit something, then hitInfo will have an index, which is used by "trace"
to get the color of the polygon it hit. If "found" is false, then there is no hitInfo,
because nothing was hit, and then "trace" returns black

float rayIntersectsTriangle(vec3 p, vec3 d, vec3 v0, vec3 v1, vec3 v2)

This function takes the origin of a ray, the direction of a ray, and the three
vertices of a triangle, to see if the ray hits the trianle, and returns the 
distance between origin and the point of intersection between the vertices (on
the triangle). The distance will always be between 0 and MAX_SCENE_BOUNDS, unless
nothing is hit, and then it returns -1, to let intersectTriangles(...) know that
the ray hit nothing, while it is checking for collisions with all the triangles.

we create several vec3s 
	vec3 e1,e2,h,s,q;
	
we create several floats
	float a,f,u,v, t;
	
float t is the one we care most about, that will determine the distance between
the ray and the triangle. When we're done doing all the math (look near the end
of the function), we check to see if 't' is more than 0.00001

0.00001 is nearest distance that we want to detect between a ray and a triangle.
This is the equivalent of the "near" variable in the projection matrix that
we've had in previous tutorials: glm::persepctive(fov, aspect, near, far).

If 't' is less than this, then either a polygon is too close, or the polygon
was not hit, so we return -1 to indicate a miss.

Prior to calculating distance, we have several other checks to see if 
a ray missed the points, there are three other checks to determine
if the ray missed, where the function will return -1. The comments
will explain how we detect intersection with the triangle. However,
if the comments are not enough to understand how this works, this
particular function is not critical to understand, we will never
change or modify this function anyways.

How to improve:
Add lights to the scene (next tutorial).
