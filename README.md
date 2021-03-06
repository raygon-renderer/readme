![Raygon Logo][logo] Raygon
===========================

## Announcements

* [This Month in Raygon 1](./blogs/month-1.md)

# Raygon

Raygon is a WIP high-performance proprietary CPU path tracer written in the Rust programming language. It will feature state of the art light transport integrators, including path tracing, bidirectional path tracing and VCM. Perhaps even more in the future.

When released, the pricing will be significantly cheaper than competing products, and I may make the source code available to indie studios or higher. It will also feature a free version for learning<sup>[1](#f1)</sup>. Plugins will of course be open source and freely available for as many programs I can write them for.

[Join our Discord Server](https://discord.gg/Y54gQxH) and consider supporting Raygon on [Patreon](https://www.patreon.com/raygon) or [Ko-Fi](https://www.ko-fi.com/raygon). Without your support, this project will not exist. Donating just $30 will give you a lifetime personal license, and $50 will get your name featured in the executable itself.

I try to keep my Discord up to date with what I'm doing and encourage feedback.

### Example

Here are some recent rendered images created simply for the purposes of testing. Their quality is not indicative of the final product, but it shows how far along the project is.

Latest renders:
![Demo 3][latest_demo3]

Extreme stress-test for object instancing with over **80 Billion** effective triangles, and also using an HDRi with environment importance sampling. Some of the tree leaves also used alpha-masked materials for greater detail.
![Demo 1][latest_demo1]

Here is a similar scene with 222 instanced lights (each an Icosahedron with 80 triangles) and about 21,891,323 total instances of grasses and trees. There are about 17,600 total triangle lights.

After recent memory optimizations, this scene uses only 4.5GB to render.
![Demo 4][latest_demo4]

![Demo 2][latest_demo2]

Example showing textured blackbody emission. With spectral rendering, we can use real physical equations to model blackbody radiation, rather than some RGB approximation.
![Demo 5][latest_demo5]

Signed Distance Field Mandelbulb animation:
![SDF Fractal][fractal2]

# Current Features

* [Performance](#performance)
* [Color](#color)
* [Materials](#materials)
* [Film](#film)
* [Camera](#camera)
* [Primitives](#primitives)
* [Lighting](#lighting)
* [Animation](#animation)
* [Planned Features](#planned-features-not-extensive)

### Performance

Much effort has been put into creating the fastest code possible. Raygon uses a custom hand-written linear algebra library based on `packed_simd`, making use of explicit SIMD wherever possible. Final binaries will be precompiled for specific architectures to make full use of SSE and AVX instruction sets on modern hardware.

Currently, Raygon just uses a regular LBVH generated with a simple SAH heuristic, which achieves performance up to 28 Million rays per second on 28 threads with an AMD Threadripper 1950X, or around 1,000,000 rays/second per thread.

Furthermore, some renders which achieved around 620,000 rays per second per core on my machine achieved over 1.1 Million rays per second **per thread** on an Intel i9 9900K.

Complex scenes such as the living room demo above acheive around 13,000,000 rays per second on average.

While less than what a GPU can achieve for brute-force path tracing, Raygon will support complex light transports that simply can't be run on a GPU, making this performance advantageous over competing renderers.

### Color

Raygon features full spectral rendering via Hero Wavelength Spectrum sampling for spectral renders with little color noise. We use a recent spectal upsampling method for accurate color reproduction from RGB colors, with some additional tweaks for emissive materials. Materials and BSDFs are fully wavelength-aware.

Additional effects such as UV light, fluorescence, iridescence and diffraction are planned.

### Materials

Raygon features a lightweight virtual machine for arbitrary material shader evaluation, allowing for complex procedural materials easily on-par with Blender Cycles, and vastly outclassing other industry renderers.

The functionality provided and already implemented is based mostly on Cycles and Unreal Engine 4's material nodes. This includes a variety of unary and binary operators, transformations, vector utilities, color utilities, random number sampling, texture sampling, and color ramps. It's incredibly easy to add even more as necessary.

Furthermore, it includes a small but effective peephole optimizer for optimizing material programs. Non-trivial materials such as the color ramp example on the Buddha statue typically run in around 120 nanoseconds.

Future plans for this include rewriting the VM to be register-based with full control flow, with perhaps a scripting language for even *more* complex procedural materials.

### Film

Raygon renders to a densely packed multi-channel film, with support for over 20 render channels such as Emission, World Normal, Direct and Indirect light, Roughness, Albedo, various object and material IDs, ThreadId, and so forth.

It also supports using unique direct/indirect light channels for different light groups, allowing the separation of different light sources and the combination of such in post-production. This can be used to tweak light emission intensity and color after the render has completed. Don't like a light? Turn it off in post or change its color entirely.

### Camera

Currently, Raygon supports only a thin-lens perspective and orthographic camera capable of Depth of Field and time integration/animated transforms. It also supports a bladed aperture of my own design, rather than a perfectly circular aperture. Demo images will be available soon.

Equirectangular, Stereoscopic Equirectangular (VR) and complex realistic cameras using real-world lens descriptions are also planned. In fact, I recently discovered a paper on deriving sparse polynomials to very accurately approximate realistic lenses, but almost as fast as a thin-lens function.

In the future, it may be possible to write custom camera scripts for more complex use cases such as baking renders.

### Primitives

Raygon will include initial support for:

* Triangles/triangle meshes
* Cubic Bézier splines
* Spheres
* Disks
* Cylinders

Others may be added later, but only if they add significant advantages over the existing primitives.

Additionally, Raygon supports signed distance field shapes, allowing for infinitely detailed procedural shapes not limited to complex mathematical shapes and fractals.

#### Thin geometry

We plan to implement "thin" geometry for subsurface scattering and transmission, allowing for faster rendering of panes of glass, soap bubbles, leaves/foliage, and similar. It will probably be named "Solidify" after Blender's modifier tool which does something similar on real geometry.

### Lighting

Raygon will support three types of lights:

* Primitive area lights
    - uses primitive geometry to emit light
* Environmental lighting
    - Using a world-wide environment map (will also be procedural using the material system)
* Emissive volumes
    - Will use a point-grid method for fast rendering and convergence.

The choice to avoid explicit point lights or directional lights was mostly because they don't offer much more than the above, but introduce subtle problems with the Monte-Carlo integration in real-world scenes.

### Animation

Raygon already partially supports integrating through time in the form of animated transformations, allowing for accurate motion blur of shapes, lights and cameras. Support for motion vectors is in progress.

Future work with this involves sampling many sub-frame keyframes and using analytic solutions to motion blur such as Time Interval Ray Tracing, which can converge in a single sample!

## Planned Features (not extensive)

* Bidirectional Path Tracing and VCM (Vertex Connection and Merging)
* Path Space Regularization
* Plugins
    * Blender, Houdini, Unreal Engine 4, Amethyst, etc.
    * Provide a agnostic network API for anyone
* LMBVH4 for even faster ray tracing with SIMD
* Many modern materials such as multi-scatter GGX
* Efficient layered materials
* Material shader JIT compilation
    * Would possibly allow for Turing-complete shaders with complex control flow
* Much more.

##### Footnotes

<b id="f1">1</b> Free version will be limited to 720p resolutions, similar to how Houdini does their Apprentice version.

[logo]: ./assets/logo48.png "Raygon Logo"
[latest_demo1]: ./assets/test216.png "Latest test render"
[latest_demo2]: ./assets/test173.png "Latest test render"
[latest_demo3]: ./assets/denoised3.png "Latest test render"
[latest_demo4]: ./assets/test251_denoised_wsky_upscaled_final.png "Latest test render"
[latest_demo5]: ./assets/test247_denoised.png "Latest test render"
[fractal2]: ./assets/fractal2.gif "Fractal"