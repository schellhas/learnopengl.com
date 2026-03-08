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