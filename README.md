# UnityGLTF

Unity3D library for importing and exporting [GLTF 2.0](https://github.com/KhronosGroup/glTF/) assets.

The goal of this library is to support the full glTF 2.0 specification and enable the following scenarios:  
- Run-time import
- Run-time export
- Design-time import
- Design-time export  
- Export and import of custom extensions  

The library is designed to be easy to extend with additional extensions to the glTF specification.  

## Installation

You can install this package from git, compatible with UPM (Unity Package Manager).
1. Open `Window > Package Manager`
2. Click <kbd>+</kbd>
3. Select <kbd>Add Package from git URL</kbd>
4. Paste 
   ```
   https://github.com/prefrontalcortex/UnityGLTF.git?path=/UnityGLTF/Assets/UnityGLTF
   ```
4. Click <kbd>Add</kbd>.  
   
> **Note**: If you want to target a specific version, append `#release/<some-tag>` or a specific commit to the URL above.

The package comes with a number of samples that you can import to learn more:  
1. Open `Window > Package Manager`
2. Select `UnityGLTF`
3. Select `Samples` and import the desired ones.

## Compatibility

The best results for import and export workflows with material extensions can be achieved on Unity 2021.3.8f1+ with URP.  

**Recommended:**  
- Unity 2021.3+
- Linear colorspace
- URP or BiRP

**Supported:**  
- Unity 2020.3+
- Linear colorspace
- URP or BiRP

**Legacy:**  
These configurations have been working in the past. They will not be updated with material extensions or new features. Also, issues in these configurations will most likely not be addressed if they're not also happening on later versions.   
- Unity 2018.4–2019.4
- Gamma colorspace

> **Note:** Issues on non-LTS Unity versions (not on 2020.3, 2021.3, 2022.3, ...) will most likely not be addressed. Please use LTS (Long-Term Support) versions where possible.  

## Current Status

UnityGLTF hasn't received official support since early 2020. However, a number of forks have fixed issues and improved several key areas, especially animation support,export workflows, color spaces and extendibility. These forks have now been merged back into main so that everyone can benefit from then, and to enable further work.  

A separate glTF implementation for Unity, [glTFast](https://github.com/atteneder/glTFast), is on its path towards becoming feature complete.  
At the time of writing glTFast has better _import support_ and UnityGLTF has better _export support_.   

glTFast and UnityGLTF can coexist in the same project; you can for example use glTFast for import and UnityGLTF for export, where each of these shine. For imported assets, you can choose which importer to use with a dropdown. glTFast import has precedence if both are in the same project.   

Key differences of glTFast include:
- It leverages modern Unity features such as Burst and Jobs and thus has better performance in some cases
- Better import support: allows importing compressed textures and meshes
– Slightly better coverage of the glTF Spec: texture transforms for other textures than baseMap and occlusionMap are supported
- HDRP import support. 

> **TL;DR:**
> - UnityGLTF has very good export support (runtime, editor, animations, materials).  
> - glTFast has better general import support (compressed meshes and textures, HDRP support).  
> - If you're using custom extensions, UnityGLTF might still be the right choice for import for its extensibility.    

## Supported Features
The lists below are non-conclusive and in no particular order. Note that there are gaps where features could easily be supported for im- and export but currently aren't. PRs welcome!

### Import and Export

- Animations, Skinned Mesh Renderers, Blend Shapes
- Linear and Gamma colorspace support
- Vertex Colors
- Cameras
- KHR_lights_punctual (Point, Spot, Directional Lights)
- KHR_texture_transform (limitation: not full flexibility for transforms per texture)
- KHR_materials_unlit
- KHR_materials_transmission
- KHR_materials_volume
- KHR_materials_ior
- KHR_materials_emissive_strength
- KHR_materials_iridescence
- KHR_materials_clearcoat
- KHR_materials_sheen (partial support)
- KHR_materials_specular (partial support)

### Export only

- Sparse accessors for Blend Shapes
- Timeline recorder track for exporting animations at runtime
- Lossless keyframe optimization on export (animation is baked but redundant keyframes are removed)
- All 2D textures can be exported, no matter if readable or not, RenderTextures included
- KHR_materials_emissive_strength
- URP materials
- (partial) HDRP materials
- glTFast materials

### Import only

- MSFT_LODExtension

## Material Extension Worfklows

To leverage the extended material model of glTF in Unity, use the `UnityGLTF/PBRGraph` material.  
It allows the use of various glTF material extensions for import, export, and inside Unity.

### Configure for Refractive Materials (Transmission and Volume)

### URP
To see transmissive and volume materials correctly in Unity, 
1. Select your URP Renderer Asset
2. Under the "Renderer Features" section, add a "Opaque Texture (Rough Refraction)" feature
3. On a PBRGraph material, check "Enable Transmission" and optionally "Enable Volume"
4. Change the Transmission,  Thickness, Index of Refraction and Attenuation values as desired.

### Built-In
To see transmissive and volume materials correctly in Unity, 
1. Make sure you have the PostProcessing v2 installed (can be installed via Package Manager).
2. 

### Material Conversions

UnityGLTF contains helpers to make converting to UnityGLTF/PBRGraph easy.  
When you switch a material from any shader to PBRGraph, an automatic conversion can run to bring the properties over in the best possible way.  
Some shaders already come with automatic conversions:
- `Standard`
- `URP/Lit`
- `URP/Unlit`

When a shader doesn't have a converter yet, UnityGLTF will ask if you want to create a *Conversion Script*.  These scripts contain all properties of the source shader and the target shader, but no specified mapping yet (as that dependends on the intent of the shader author).  
After the conversion script has been created, you can edit it to correctly map from the source shader's properties to PBRGraph properties.   
When you switch such a shader to PBRGraph the next time, your conversion script will run and automatically translate the materials in the specified way.  

> **Note:** Currently, conversion scripts aren't used automatically on glTF export. Convert materials at edit time for best results. 

## Material and Shader Export Compatibility

If you want to design for glTF export, it's recommended to use Unity 2021.3+ with URP and the **UnityGLTF/PBRGraph** material. It comes with support for modern material extensions like refraction and iridescence, and allows for perfect roundtrips. Great for building glTF pipelines in and out of Unity.  

| Render Pipeline | Shader | Notes | Source | 
| - | - | - | - | 
| URP on 2020.3+<br/>Built-In on 2021.3+ | **UnityGLTF/PBRGraph** <br/>☝️ *Use this if you're not sure* | Perfect roundtrip, Material Extensions | UnityGLTF |
| | UnityGLTF/UnlitGraph | Perfect roundtrip | UnityGLTF |
| | ShaderGraphs/glTF-pbrMetallicRoughness ||  glTFast |
| | ShaderGraphs/glTF-unlit | | glTFast |
| URP | URP/Lit | | Unity |
| | URP/Unlit | | Unity |
| Buit-In | Standard | Unity |
| | GLTF/PbrMetallicRoughness | | UnityGLTF (legacy) |
| | GLTF/Unlit | | UnityGLTF (legacy) |
| | glTF/PbrMetallicRoughness | | glTFast (legacy) |
| | glTF/Unlit ||   glTFast (legacy) |
| HDRP (limited support) | HDRP/Lit ||  Unity |
| | HDRP/Unlit ||  Unity |

### Legacy
These extensions and shaders worked in the past but are not actively supported anymore.  

- KHR_materials_pbrSpecularGlossiness  
  *This extension has been deprecated by Khronos. Please use pbrMetallicRoughness instead.*  
  - GLTF/PbrSpecularGlossiness (legacy shader from UnityGLTF)
  - ShaderGraphs/glTF-pbrSpecularGlossiness (legacy shader from glTFast)
  - glTF/PbrSpecularGlossiness (legacy shader from glTFast)

## Extensibility

There's lots of attachment points for exporting and importing glTF files with UnityGLTF. These allow modifying node structures, extension data, materials and more as part of the regular export and import process both in the Editor and at runtime.  

> 🏗️ Under construction. You can take a look at MaterialExtensions.cs for an example in the meantime.  

## Animation Export

Animations can be exported both in the Editor and at runtime. 

### Animator Controller

You can export entire Animators and their clips as glTF files with multiple animations.  
Animation clips will be named after each Motion State in the Animator Controller.  
The "speed" property of each Motion will be baked into the exported clip. Ensure the speed is 1 when you want to export unchanged.    
Any number of Animators in a hierarchy is supported, as is any number of clips in those.  

Both Humanoid and Generic animations will be exported. Humanoid animations are baked onto the target rig at export time.  

> **Note**: Animator export only works in the Editor. For runtime export, use the GLTFRecorder capabilities or the Timeline Recorder.  

### GLTFRecorder

For creating and/or recording animations at runtime, you can use the GLTFRecorder. It allows to *capture* the state of entire hierarchies and complex animations and export them directly as glTF file, with or without KHR_animation_pointer support.  
An example is given as GLTFRecorderComponent.  

### Timeline Recorder  

Timelines or sections of them can be recorded with a 

### Legacy: Animation Component

> **Note**: Animator export only works in the Editor. For runtime export, use the GLTFRecorder capabilities.  

Animation components and their legacy clips can also be exported.  

## Animation Import

Animations can be imported both in the Editor and at runtime.  
On the importer, you can choose between "Legacy" or "Mecanim" clips.  

At runtime, if you're importing "Mecanim" clips, you need to make sure to add them to a playable graph (e.g. Animator Controller or Timeline) to play them back.  

## Blendshape Export

Morph Targets / Blend Shapes are supported, including animations.  
To create smaller files for complex blendshape animations (e.g. faces with dozens of shapes), you can export with "Sparse Accessors" on.  

> **Note**: While exporting with sparse accessors works, importing blend shapes with sparse accessors is currently not supported.  

## Known Issues

Each known issue can be reproduced from a specific [glTF Sample Model](https://github.com/KhronosGroup/glTF-Sample-Models/tree/master/2.0):

- khronos-SimpleSparseAccessor: sparse accessors not supported on import (can be exported though)  
- khronos-TriangleWithoutIndices: meshes without indices import with wrong winding order  
- khronos-MultiUVTest: UV per texture is imported but not supported in the GLTF-Builtin shader  
- khronos-MorphPrimitivesTest: isn't correctly importing at runtime (in some cases?)  
- khronos-NormalTangentTest: import results don't match expected look  

## Contributing

This section is dedicated to those who wish to contribute to the project. 

<details>
<summary>More Details</summary>

### [GLTFSerializer](https://github.com/KhronosGroup/UnityGLTF/tree/master/GLTFSerialization)

- **Basic Rundown**: The GLTFSerializer facilitates serialization of the Unity asset model, and deserialization of GLTF assets.  

- **Structure**: 
	- Each GLTF schemas (Buffer, Accessor, Camera, Image...) extends the basic class: GLTFChildOfRootProperty. Through this object model, each schema can have its own defined serialization/deserialization functionalities, which imitate the JSON file structure as per the GLTF specification.
	- Each schema can then be grouped under the GLTFRoot object, which represents the underlying GLTF Asset. Serializing the asset is then done by serializing the root object, which recursively serializes all individual schemas. Deserializing a GLTF asset is done similarly: instantiate a GLTFRoot, and parse the required schemas.

### [The Unity Project](https://github.com/KhronosGroup/UnityGLTF/tree/master/UnityGLTF)

- **Unity Version**
	Be sure that the Unity release you have installed on your local machine is *at least* the version configured for the project (using a newer version is supported). You can download the free version [here](https://unity3d.com/get-unity/download/archive). You can run this project simply by opening the directory as a project on Unity.
- **Project Components**
	The Unity project offers two main functionalities: importing and exporting GLTF assets. These functionalities are primarily implemented in `GLTFSceneImporter` and `GLTFSceneExporter`.

### Tests

To run tests with UnityGLTF as package, you'll have to add UnityGLTF to the "testables" array in manifest.json:

```
"testables": [
	"org.khronos.unitygltf"
]
```

</details>

## Samples

1. Add the package to your project as described above
2. Open Package Manager and select UnityGLTF
3. Import any sample.