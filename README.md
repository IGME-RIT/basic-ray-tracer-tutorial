# Basic Ray Tracer Tutorial

This program serves to demonstrate the concept of ray tracing.

# About

This example is very basic, storing the triangles as a hardcoded constant in the Fragment Shader itself. It draws a quad with the Vertex Shader, and renders each pixel via tracing a ray through that pixel from the camera position. There is no lighting.

# Setup

In order to setup, run the following in a shell, then open the project in your preferred editor. Windows setup has been configured for use with Visual Studio.

Windows:
```
cd path/to/folder
setup.cmd
```
Linux:
```
cd path/to/folder
./setup
```

# Credits

For more reading, check out [ray tracing with compute shaders](https://github.com/LWJGL/lwjgl3-wiki/wiki/2.6.1.-Ray-tracing-with-OpenGL-Compute-Shaders-(Part-I))
