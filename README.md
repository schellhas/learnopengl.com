# Part 1 - Getting started

## OpenGL

opengl is a specification of an API maintained by khronos, the implementation is usually done by graphics card manufacturers

### immediate mode and core-profile

- immediate mode was historically used more, easier to use, less power/flexibility, abstracts a lot
- nowadays we basically only use core-profile, more powerful, flexible, efficient, more tedious to implement

### extensions

- extensions are a way to use new and modern ways graphics card manufacturers found to do a thing, while still providing an old way of doing it for all the cards that dont support it yet

```cpp
if(GL_ARB_extension_name) {
	do_it_cool();
} else {
	do_it_old();
}
```

### state machine

- opengl is a state machine
- collection of variables decide how opengl should currently operate
- this state is usually referred to as opengl context
- we often change the opengl context by setting some options, manipulation some buffers and then render using the current context
- e.g. we change some context variable that decides wether we draw lines or triangles. after we changed it, it will now draw lines instead of triangles
- there are "state-changing" functions, they change the context
- and there are "state-using" functions, they perform operations based on the current state of opengl
- keep in mind, opengl is a large state machine, this way most functionality will make sense

### objects

- opengl libraries are written in C. one of the most important asbtractions from the libraries are "objects".
- an "object" in opengl is a collection of options that represent a subset of opengls state.
- example: an object represents the setting of drawing a window. we can set the windows size, how many colors it supports, etc.

```cpp
struct object_name {
	float	option1;
	int	option2;
	char[]	name;
};
```

- Whenever we want to use objects it generally looks something like this (with OpenGL’s context visualized as a large struct):

```cpp
// The state of OpenGL
struct OpenGL_Context {
	...
	object_name* object_Window_Target;
	...
};
```

```cpp
// create object
unsigned int objectId = 0;
glGenObject(1, &objectId);
// bind/assign object to context
glBindObject(GL_WINDOW_TARGET, objectId);
// set options of object currently bound to GL_WINDOW_TARGET
glSetObjectOption(GL_WINDOW_TARGET, GL_OPTION_WINDOW_WIDTH, 800);
glSetObjectOption(GL_WINDOW_TARGET, GL_OPTION_WINDOW_HEIGHT, 600);
// set context target back to default
glBindObject(GL_WINDOW_TARGET, 0);
```

- this is a typical opengl workflow
- first create an object and store a reference to it as an id
- then bind the object, using the id, to the target location of the context

## Creating a window

- creating a window is specific to the OS so opengl doesnt do it. creating window, define a context (?) and user input is done by us not opengl
- most libraries that do that are OS independent, some are GLUT, SDL, SFML and GLFW. learnopengl.com uses GLFW

### glfw

- glfw is very bare. creates an opengl context, defines window parameters, handles user input
- use on debian:

```cpp
#include <GLFW/glfw3.h>
```

> some helpful compiler flags: `-lglfw3 -lGL -lX11 -lpthread -lXrandr -lXi -ldl`

### glad

- since opengl is just a standard/specification, the driver manufacturer has to implement the specification that the specific graphics card supports
- the location of most of this drivers function is not known at compile time and needs to be queried at run time
- you would need to do this for each function manually
- glad does that
- on glad.dav1d.de
- Profile: Core
- API gl: Version above 3.3
- tick the box "Generate a loader"
- click generate
- copy include directory to your project
- copy glad.c into your project

```cpp
#include <glad/glad.h>
```

- compiling now shouldnt give any errors

compile with: `g++ main.cpp src/glad.c -Iinclude -lglfw -lGL -lX11 -lpthread -lXrandr -lXi -ldl`

## Hello window

> glad has to be included before glfw

### glad

in the same way glad has to be included before glfw, glad has to be initialized before opengl, to manage the function pointers

### viewport

we tell opengl the window dimensions so it can properly calculate and use them with respect to the window. to also adjust it when the user resizes it, theres *callback functions*.

```cpp
void framebuffer_size_callback(GLFWwindow* window, int width, int height);
```

to tell GLFW we actually want to call this function each time we resize the window, we say:

```cpp
glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);
```

### render loop

- ```glfwWindowShouldClose``` checks each iteration wether GLFW has been instructed to close
- ```glfwPollEvents``` checks if events are triggered (keyboard/mouse), updates window state, calls corresponding functions
- ```glfwSwapBuffers``` swaps color buffer (a large 2D buffer that contains color vlaues for each pixel in GLFWs window) that is used to render to during this render interation and show it as output to the screen

### terminate

as soon as we exit the render loop, clean all GLFW resources we've allocated via ```glfwTerminate```

### input

theres several GLFW functions for input. we'll use ```glfwGetKey``` that takes the window as an input together with a key.the function returns wether this key is currently being pressed.

## Hello Triangle

to transform 3d coordinates to 2d pixels (screen/window is one large array of pixels), we use the "graphics pipeline". the graphics pipeline can be divided into two large parts: transforming 3d coordinates to 2d coordinates and transforming 2d coordinates to actual colored pixels.

the graphics pipeline is divided into several small steps where each takes the output of the previous one as input. they can be processed in parallel nicely. Because of their parallel nature, graphics cards of today have thousands of small processing cores to quickly process your data within the graphics pipeline. The processing cores run small programs on the GPU for each step of the pipeline. These small programs are called "shaders".

we can configure some of these shaders to replace the default shaders. this way we get more fine-grained control with the usual pros and cons. shaders are written in the OpenGL Shading Language (GLSL).

`vertex_data[]` -> Vertex Shader (configurable) -> Geometry Shader (configurable) -> Shape Assembly (not configurable) -> Rasterization (not configurable) -> Fragment Shader (configurable) -> Tests and Blending (not configurable)

### input

as input to the graphics pipeline we pass a list of three 3d coordinates that should form a triangle in an array here called `vertex_data`, this vertex data is a collection of vertices. A "vertex" is a collection of data per 3d coordinate. this vertex's data is represented using "vertex attributes" that can contain any data we'd like, but for simpllicity's sake we assume that each vertex consists of just a 3d position and some color value.

> We tell OpenGL with "primitives" how to use our vertex data. We can also call them "hints" and some of them are: `GL_POINTS`, `GL_TRIANGLES`, `GL_LINE_STRIP`.

### vertex shader (1st part of graphcis pipeline)

takes as input a single vertex. it transforms 3d coordinates into different 3d coordinates (lol, more later), and the vertex shader allows us to do some basic processing on the vertex attributes (what type of processing?).

### geometry shader (optional)

takes as input a collection of vertices that form a primitive and has the ability to generate other shapes by emitting new vertices to form new (or other) primitives. in this example case, it generates a second triangle out of the given shape.

### primitive assembly

takes as input all the vertices (or vertex of `GL_POINTS` is chosen) from the vertex or geometry shader that form one or more primitives and assembles all the points in the primitive shape given. in this case two triangles.

### rasterization stage

output of primitive assembly stage is passed to rasterization stage. it maps the resulting primitives to the corresponding pixels on the final screen, resulting in fragments for the fragment shader to use. before the fragemnt shaders run, "clipping" is performed. clipping discards all fragemtns that are outside your view, increasing performance.

> A fragment in OpenGL is all the data required for OpenGL to render a single pixel.

### fragment shader

calculates the final color of a pixel and this is usually the stage where all the advanced OpenGL effects occur. usually the fragemnt shader contains data about the §d scene that it can use to calculate the final pixel color (like lights, shadows, color of the light and so on).

### alpha testing and blending

checks the corresponding depth of the fragemtn and uses those to check if th eresulting fragment is in front or behind other objects and should be discarded accordingly. the stage also checks for "alpha" values (alpha values define the opacity of an objects) and "blends" the objects accordingly. so even if a pixel output color is calculated in the fragemnt shader, the final pixel color could bstill be something entierly different when rendering multiple triangles.

### vertex and fragment shader

in modern opengl we have to define at least a vertex and fragment shader of our own.

### vertex input

to draw something we have to give OpenGL some input vertex data. all coordinates that we specify are 3D (x,y,z). opengl only processes coordinates between -1.0 and 1.0. all coordinates in this "normalized device coordinates" range will end up visible on the screen. (all outside it wont)

we want to render a triangle, so we need three vertices. each vertex has a 3d position. we define them in normalized coordinates in a `float` array:

```cpp
float vertices[] = {
	-0.5f, -0.5f, 0.0f,
	0.5f, -0.5f, 0.0f,
	0.0f, 0.5f, 0.0f
};
```

we keep the z coordinates at 0.0, so it looks 2d, (one plane stays the same).

