# Primal framework Sponza showcase (WIP)
![alt text](https://github.com/callmeEthan/PrimeFramework_Sponza/blob/d6bf7f9768fba0e97f2d7055d827bb69d3eb70b2/Screenshots/header.jpg?raw=true)
This is my demo to showcase Primal framework, this engine is build in gamemaker 2. Intended for 2.5d or 3d games with pixel art style, it use deferred rendering with various post processing effect. This demo is compiled with YCC for windows.  

**Upon load, it might take a few minutes to load the scene, when it finish you can use mouse to look around, and WASD keys to move the camera.
To tweak the graphic setting, press 2 to equip graphic controller tool, then right click to open options dialog. Click and hold any input option and drag the mouse left or right to adjust the value automatically.**  

It also include a map editor, press 1 to equip. You can use it to place light sources, 3d models, decals, foliages and smoke effect in the scene. While placing object, press and hold control node to move object around, right click and hold any where to control camera direction, middle click and hold to move camera relatively.  

# Deferred rendering effect
I used [**TheSnidr's deferred rendering**](https://thesnidr.itch.io/) as basis to learn. Most of the code is now replaced and or improved upon, with various new effect I learned/came up with.  
Deferred rendering is used with basic color/normal/specular map (not PBR :(), Scene are encoded to 3 surfaces using MRT, containing color, normals, fragment type, depth, specular, roughness, scattering and emissive.  

### Directional shadow
Use a single shadow map to cover the entire area, utilize vertex transform to squeeze distance objects inside map while trying to maintain shadow quality near the camera.  
However there are draw back to this method as vertex transform are not perfect if the mesh triangle is too large. So larger triangle need to be broken down to smaller triangles, this is why the scene took longer time to load.  
**Image before and after vertex transform:**
<img src="https://github.com/callmeEthan/PrimeFramework_Sponza/blob/main/Screenshots/shadow_near.jpg?raw=true" width="1024">
<img src="https://github.com/callmeEthan/PrimeFramework_Sponza/blob/main/Screenshots/shadow_far.jpg?raw=true" width="1024">

### Pointlight spotlight
There is also 3d pointlight/spotlight with shadow mapping and deferred pointlight/spotlight with no shadow map. Though there is nothing much notable to say about them. You can use the map editor tool to try placing them around.  

### Global Illumination
This is based on [2d luminance model by petercowal](https://github.com/petercowal/luminance-model), modified for 3d scene.  
It voxelize the scene by using a camera trick and vertex transform (since gamemaker don't support geometry shader), however this method is quite slow as it only render one layer at a time.  
Voxel scene and signed-distance-field map (SDF) are stored in a 2d surface, and combined to create a 3d sdf+color map. Then, as usual, cast ray in all direction and interpolate color result.   
There are many draw back however:
- The process is quite slow, heavy temporal filter is applied to hide color flickering.
- You can force it to render multiple layer to boost light process speed, but it can be costly on performance.
- Due to the nature of voxel, lighting can be inaccurate if the voxel scale is too big, light can leak through wall.  

This is only suitable for outdoor scene, where lighting doesn't need to be accurate.  
**Light map after being processed:**
<img src="https://github.com/callmeEthan/PrimeFramework_Sponza/blob/main/Screenshots/Screenshot%20(29).jpg?raw=true" width="1024">
**Image comparision:** No GI, with GI and Light only
<img src="https://github.com/callmeEthan/PrimeFramework_Sponza/blob/main/Screenshots/gi_comparision.jpg?raw=true?raw=true" width="1024">
Inaccurate light sampling
<img src="https://github.com/callmeEthan/PrimeFramework_Sponza/blob/main/Screenshots/gi_comparision2.jpg?raw=true?raw=true" width="1024">
An outdoor scene without GI and with GI
<img src="https://github.com/callmeEthan/PrimeFramework_Sponza/blob/main/Screenshots/gi_comparision3.jpg?raw=true?raw=true" width="1024">
### Reflection
Screen space reflection is mixed with SDF raycasting (using sdf map generated in Global Illumination),

