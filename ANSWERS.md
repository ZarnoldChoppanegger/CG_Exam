## Describe the Phong illumination model, and explain what effect the
various components of this model have on the appearance of a surface.

The Phong Illumination model is a so-called “local illumination model”. This means that the illumination of a surface will depend solely on the viewer perspective, the light source and the material properties of the surface.
The Phong illumination model, on a single point, can be described by the following equation:

I = K_a * I_a + K_d * I_d * max(V * L, 0) + K_s * I_s * (V * H)^\alpha

The components of the model are: 
- ambient (K_a * I_a)
- diffuse (K_d * I_d * max(L * N, 0))
- specular (K_s * I_s * (V * H)^\alpha)

The ambient component is supposed to give a rough approximation of an important part of light behavior that is not represented by the model (i.e. the indirect light that is reflected from other surfaces in the scene). Without this, points that are not directly illuminated by the light source would be in complete darkness.

The diffuse component represents the directional impact of light on a surface. That is, the more a surface faces the light source, the brighter it will appear. The dot product between the light ray (L) and the normal (N) represents the angle between the two vectors. The higher the angle, the lower the light intensity on the surface will be. At 90 degrees angle, the light contribution will be zero.

The specular component simulates the bright spot that appears on shiny objects (e.g. metal, plastic). This component depends on the light's direction,  surface normal and viewer's direction. The vector H is called the halfway vector and is computed by reflecting the light's direction on the normal. The angle between the halfway vector and the viewer's direction will determine the impact of the specular light. The term \alpha determines the shininess of the object. The higher it is, the smaller and intense the highlight of the reflection becomes.
## Describe the Goraud and Phong shading models, and discuss the visual difference between the two models.

Both models are involved in the process of "coloring" an object based on components like light position, surface material properties, camera position (and more).

The Goraud shading model goes as follows:
compute the surface normal at each polygon vertex
apply the phong illumination model at those vertices to get the color
using linear interpolation, get the color for every point across the edges of the polygon and then across the scan-lines
The Phong shading model:
compute the surface normal at each polygon vertex
using linear interpolation, compute the normals for every point across the edges of the polygon and then across the scan-lines
apply the phong illumination model to every point in the polygon using their respective normals
Goraud is computationally faster than Phong shading, because the illumination model is applied only at the vertices. On the other hand, Goraud doesn't give believable results when dealing with specular reflections that generate intense highlights. On the other hand, Phong gives much better results with the downside of being more costly.

## Describe the following two algorithms for hidden surface removal, and discuss their respective advantages and disadvantages:

### Painter's algorithm
It can be summarized in the following steps:
sort the elements of the scene based on their distance from the viewport
draw the elements from the furthest to the nearest
### Z-buffer algorithm
create a buffer (z-buffer) with size equal to the image (frame buffer)
for every point that is encountered in the scene, compare its depth value with the corresponding one in the z-buffer
if and only if the depth value is less than the one in the z-buffer, then draw the point and write the new depth value into the z-buffer
### Advantages and disadvantages
Painter's algorithm advantages: low memory overhead, but certain scenarios (like intersecting polygons) cannot be drawn correctly.
Z-buffer algorithm: simple implementation, does not require sorting, and can be implemented in hardware. The problem with this approach is precision. If the near and far clipping planes are too distant from each other, it may give wrong results.
## Explain the concepts of local illumination and global illumination. Give
examples of visual phenomena that are more easily achieved with global
illumination.

Local illumination: only light source, viewer's direction, and surface properties.
Indirect light that is reflected from nearby objects is not taken into account.
It misses an important behavior of light that is roughly approximated (e.g. the ambient component in the Phong model).

Global illumination models try to account for indirect light, by solving the rendering equation. Even though this is computationally more intensive, it achieves better results. By modelling the behavior of light effects like shadows, environment mapping, color bleeding, refraction, are generated automatically in the process of solving the rendering equation.

## Imagine that you are implementing ray tracing for rendering scenes consisting of large numbers of polygons (triangles). Describe two different approaches for accelerating the ray tracing algorithm in this scenario.

The basic idea of accelerating ray tracing when many polygons present, is to reduce the  number of ray intersection checks that need to be performed. Some techniques are as follows:

*Bounding volumes*: enclose the complex mesh in a simpler geometry, like a cube or sphere, and check if the ray first intersects with that one rather than with every polygon in the mesh.
*Space partitioning*: divides the space in disjoint subsets. The methods can be k-d trees, octtrees or bsp trees. The ray intersection is performed only with the polygons that are in the same region.
## Describe the basic idea and purpose of texture mipmapping.

Many times, a pixel does not correspond to a texel. We then have two scenarios:
*Magnification*: a texel is represented by many pixels.
*Minification*: a pixel contains many texels.
The latter case will introduce *aliasing* problems.
Mipmapping solves this problem by filtering down the texture at different levels, in smaller images. During rendering, for distant objects, textures with high mipmap levels will be selected, and vice versa for closer objects.

## Explain how bump or normal mapping can be used to add detail to a model

Bump and normal mapping add detail to a model by perturbing its surface normals. In this way, when lighting is applied to the model, it will give the illusion that the geometry is more detailed than what it actually is. So, compared to other methods like /surface mapping/, where the vertices of the model are displaced, normal or bump mapping keep the geometry unchanged.

Bump mapping makes use of a grey-mapped texture containing the height values. Thus, *in the shader*, for every point it is needed to compute the surface normal. Normal mapping on the other hand uses a RGB texture containing the precomputed values of the normals. So bump mapping has less memory overhead but it's computationally more intensive.

## Derive a 4×4 matrix M, representing a transformation that mirrors all points in a plane defined by a point p = (px, py, pz) and the normal vector (0, 1, 0).

We know that the normal in the y axis, thus if we want to take any point on the plane to the plane centered in the origin, we need to translate by -p_y.

       1 0 0 p_x
T=  0 1 0 -p_y
       0 0 1 p_z
       0 0 0  1

Then to mirror any arbitrary point around the plane, we use the following matrix:

        1 0 0 0
M = 0 -1 0 0
        0 0 1 0
        0 0 0 1

Then we translate back to the original reference by applying T^-1. 

Given an arbitrary point p, mirroring around the plane will look like:  T^-1 * T * M * p

## Compute a model-view matrix M that positions the camera at the point
(1, 1, 0), looking towards the origin.

First let's define the *y* vector coordinate as (0, 1, 0).
Now, the camera frame of reference will be defined by the following three orthogonal vectors: direction, right, up.

direction = normalized(position - origin)
right = normalized(cross(direction, y))
up = normalized(cross(right, direction))

The final matrix that will position the camera is:

right_x           right_y           right_z           0                   1 0 0 -p_x
up_x               up_y                up_z               0       *          0 1 0 -p_y
direction_x  direction_y  direction_z  0                   0 0 1 -p_z
0                        0                       0                      1                   0 0 0     1

## Derive the coefficients for a third degree polynomial curve that interpolates four given control points p0, p1, p2, p3. (Note: You do not need to provide the exact numeric values for the coefficients, just a derivation of how they can be calculated. Specifically, you do not need to perform any explicit matrix inversion

p(u) = c_0 + c_1u + c_2u^2 + c_3u^3

p(0) = c_0
p(1/3) = c_0 + c_1(1/3) + c_2(1/3)^2 + c_3(1/3)^3
p(2/3) = c_0 + c_1(2/3) + c_2(2/3)^2 + c_3(2/3)^3
p(1) = c_0 + c_1 + c_2 + c_3

Let p be a vector, and c = [c_0, c_1, c_2, c_3]. Let M be the matrix
1     0        0          0
1 1/3 (1/3)^2 (1/3)^3     
1 2/3 (2/3)^2 (2/3)^3
1     1       1           1

so p = M * c

## Describe the basic idea behind volume rendering with GPU-accelerated ray-casting

Given a 3d space containing voxels (gathered from real life scans or procedurally generated), cast rays through the volume and sample information along the way (i.e. color, intensity etc...). Average the sampled information and display it.

## A Gaussian filter smooths an image using convolution with a 2D filter kernel. Briefly describe (in 3-4 steps) how you would implement this type of 2D image filter as a post-processing effect, using the programmable graphics pipeline and render-to-texture functionality.

Post-processing effects require to use user defined frame buffers (FBOs) instead of the default one. Applying a post-processing effect can be done in the following way:
- render the scene on a texture
- bind the texture to a FBO
- define a quad and map the texture to it
- in the fragment shader apply the desired effect, in this case the convolution of the Gaussian filter
- unbind the FBO and render the scene
