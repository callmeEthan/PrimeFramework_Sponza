# Primal framework Sponza showcase
![alt text](https://github.com/callmeEthan/PrimeFramework_Sponza/blob/d6bf7f9768fba0e97f2d7055d827bb69d3eb70b2/Screenshots/header.jpg?raw=true)
This is my demo to showcase Primal framework, this engine is built in gamemaker 2, intended for 2.5d or 3d games with pixel art style. It use deferred rendering with various post processing effects. Most shaders are written in GLSL ES.  
This demo is compiled with YYC for windows.  
[<img src="https://github.com/callmeEthan/PrimeFramework_Sponza/blob/main/Screenshots/ads.jpg?raw=true">](mailto:name_Ethan@outlook.com)

**Upon load, it might take a few minutes to load the scene, when it finish you can use mouse to look around, and WASD keys to move the camera.  
To tweak the graphic setting, press 2 to equip graphic controller tool, then right click to open options dialog. Click and hold any input option and drag the mouse left or right to adjust the value automatically.**  

It also include a map editor, press 1 to equip. You can use it to place light sources, 3d models, decals, foliages and smoke effect in the scene.  
While placing object, press and hold control node to move object around, right click and hold any where to control camera direction, middle click and hold to move camera relatively.  

This engine is very much a **work in progress**, with more effect to be added or removed, **Some configuration option don't actually do anything**.  

# Deferred rendering effects
I used [**TheSnidr's deferred rendering**](https://thesnidr.itch.io/) as basis to learn. Most of the code is now replaced and or improved upon, with various new effect I learned/came up with.  
Deferred rendering is used with basic color/normal/specular map (not PBR :disappointed:). Scene are encoded to 3 surfaces using MRT, containing color, normals, fragment type, depth, specular, roughness, scattering and emissive.    

### Directional shadow
Use a single shadow map to cover the entire area, utilize vertex transform to squeeze distance objects inside map while trying to maintain shadow quality near the camera.  
However there are draw back to this method, as vertex transform are not perfect if the mesh triangle is too large. Larger triangle need to be broken down to smaller triangles, this is why the scene took longer time to load.  
**Image before...**
<img src="https://github.com/callmeEthan/PrimeFramework_Sponza/blob/main/Screenshots/shadow_near.jpg?raw=true" width="1024">
**...and after vertex transform**
<img src="https://github.com/callmeEthan/PrimeFramework_Sponza/blob/main/Screenshots/shadow_far.jpg?raw=true" width="1024">

### Pointlight & spotlight
There is also 3d pointlight/spotlight with shadow mapping and deferred pointlight/spotlight with no shadow map. Though there is nothing much notable to say about them. You can use the map editor tool to try placing them around.  
**Pointlight shadow and spotlight shadow**
<img src="https://github.com/callmeEthan/PrimeFramework_Sponza/blob/main/Screenshots/light_sources.jpg?raw=true" width="1024">
There is not much filtering/anti-aliasing to hide the shadows artifacts, however this engine is intended for pixel art style anyway. A heavy dithering filter would probably hide such flaws.

### Global Illumination
This is based on [2d luminance model by petercowal](https://github.com/petercowal/luminance-model), modified for 3d scene.  
It voxelize the scene by using a camera trick and vertex transform (since gamemaker doesn't support geometry shader), however this method is quite slow as it only render one layer at a time.  
Voxel scene and signed-distance-field map (SDF) are stored in a 2d surface, and combined to create a 3d sdf+color map. Then, as usual, cast ray in all direction and interpolate color result.   
There are many draw back however:
- The process is quite slow, heavy temporal-filter is applied to hide color flickering.
- You can increase the number of layer to cover taller scene, but this slow down light process speed significantly.
- You can force it to render multiple layer per frame to boost light process speed, but it can be costly on performance.
- Due to the nature of voxel, lighting can be inaccurate if the voxel scale is too big, light can also leak through wall.  

This is only suitable for outdoor scene, where lighting doesn't need to be accurate.  
**Image comparision:** No GI, with GI and Light only
<img src="https://github.com/callmeEthan/PrimeFramework_Sponza/blob/main/Screenshots/gi_comparision.jpg?raw=true?raw=true" width="1024">
**Incorrect light sampling**
<img src="https://github.com/callmeEthan/PrimeFramework_Sponza/blob/main/Screenshots/gi_comparision2.jpg?raw=true?raw=true" width="1024">
**Light map after being processed:**
<img src="https://github.com/callmeEthan/PrimeFramework_Sponza/blob/main/Screenshots/Screenshot%20(29).jpg?raw=true" width="1024">
**An outdoor scene without GI and with GI**
<img src="https://github.com/callmeEthan/PrimeFramework_Sponza/blob/main/Screenshots/gi_comparision3.jpg?raw=true?raw=true" width="1024">
### Reflection
Screen space reflection is mixed with SDF raycasting (using sdf map generated in Global Illumination). Use noise sampling for material roughness, and apply a temporal-filter with intensity depends on level of roughness (the blurrier/noiser it is, the heavier the TAA weight).  
SDF reflection isn't perfect due to low voxel resolution, but good enough for estimating sky mask.  
**Image: screen space reflection only vs mixed with SDF raycasting**
<img src="https://github.com/callmeEthan/PrimeFramework_Sponza/blob/main/Screenshots/reflection.jpg?raw=true" width="1024">
<img src="https://github.com/callmeEthan/PrimeFramework_Sponza/blob/main/Screenshots/reflection2.jpg?raw=true" width="1024">
### God ray
After reading [this blog](https://www.alexandre-pestana.com/volumetric-lights/), volumetric god ray is actually quite easy to implement, cast a ray from camera position to fragment position and check for directional shadow along the way.  
But large number of ray step can cost performance, instead, use lower number of step and apply a dithering pattern to offset the ray origin position. Then apply a blur filter to smooth out the image result.  
**Image: volumetric ray only, final result**
<img src="https://github.com/callmeEthan/PrimeFramework_Sponza/blob/main/Screenshots/godray1.jpg?raw=true?raw=true" width="1024">
<img src="https://github.com/callmeEthan/PrimeFramework_Sponza/blob/main/Screenshots/godray2.jpg?raw=true?raw=true" width="1024">
Ray detail are not well conceived in photo but you can see them clearer in motion.
### Smoke shadow
For particles effect, [**theSnidr's sPart 3D Particle System**](https://marketplace.gamemaker.io/assets/7299/spart-3d-particle-system) is implemented, modified to work with deferred rendering. Using depth gbuffer to create soft particle effect.  
Smoke shadow was quite tricky because the particles are order-independent, and you can't render transparency with depth buffer.  
So I apply a noise sampling instead of transparency, then process lights effect on it as if it's a normal scene. Finally, apply a heavy amount of temporal-filter/down-sampling to hide the noise.  
I also include global illumination in the light sampling to make it more realistic.  
God ray also sample particle depth to avoid overlapping.  
**Image: Particle noise sampling, final result**
<img src="https://github.com/callmeEthan/PrimeFramework_Sponza/blob/main/Screenshots/smoke_shadow1.jpg?raw=true?raw=true" width="1024">
Smoke effect blending into the environment pretty well
<img src="https://github.com/callmeEthan/PrimeFramework_Sponza/blob/main/Screenshots/smoke_shadow2.jpg?raw=true?raw=true" width="1024">
Overal result are good, however you can still see the noise if you look closely.
### Foliages
I got the idea after watching the first minute of [this video](https://www.youtube.com/watch?v=iASMFba7GeI). The grasses and tree trunks isn't complicated, but for the tree leaves and bushes I have to come up with [**Naive surface mesh**](https://github.com/callmeEthan/Naive_surface_nets_GM2) for gamemaker.  
The final result is a tree model with custom attribute for vertex transform (wind blowing).  
Draw back is that naive surface mesh can be very slow to generate, so I implement a frametime check so the process doesn't pause the game, and add the model into scene once the generation is completed.  
**Image: tree mesh before and after vertex transform**
<img src="https://github.com/callmeEthan/PrimeFramework_Sponza/blob/main/Screenshots/foliage.jpg?raw=true?raw=true" width="1024">
The mesh still have some unused attribute, which can be tweak for things like collision and stuff, probably.
## Other effects
That there is nothing notable to talk about...

**Screen space ambient occlusion:** you probably already know what it is.  
**Fog:** Apply a color using depth gbuffer.  
**Depth of field:** Bokeh blur sampling.  
**Cloud:** [**TheSnidr's** worley noise cloud](https://www.youtube.com/watch?v=SijNZQne4a4) with some tweak.  
**Bloom**  

## Benchmark
GTX 1060 might struggle to reach 50fps at full 1080p resolution, but can reach 60fps by lower lighting and effect resolution to 0.6~0.7  
Of course RTX 2060 should have no problem running at 60fps  

## Credit
[petercowal](https://github.com/petercowal) for his 2d luminance model.  
[TheSnidr](https://www.youtube.com/TheSnidr), you beautiful man.  
All credit to their respective creator for their intellectual work.  
[<img src="https://github.com/callmeEthan/PrimeFramework_Sponza/blob/main/Screenshots/ads.jpg?raw=true">](mailto:name_Ethan@outlook.com)
