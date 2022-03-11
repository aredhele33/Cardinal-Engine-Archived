# Cardinal Engine (Archived)
#### This is an archived project. It only contains a runnable binary since I don't have time to maintain the code of this project.

<p align="center">
  <img src="https://raw.githubusercontent.com/Aredhele/Cardinal-Engine-Archived/main/Press/00088_Cardinal_Engine.jpg" />
</p>

## Abstract
Cardinal Engine (Archived) is an experimental 3D game engine programmed in one month for a school project at the 
<a href="https://enjmin.cnam.fr">**Enjmin**</a>, the French National School of Video Games. The main goal of the project
was about learning about 2D, 3D rendering and procedural generation by creating a Minecraft like world from scratch.

Cardinal Engine (Archived) is fully written in C++ using OpenGL 3 and use some third party libraries :
(Some links could be unavailable)

| Name                                                    | Used for                                   |
|---------------------------------------------------------|--------------------------------------------|
| [Glew](https://github.com/nigels-com/glew)              | OpenGL loader                              |
| [Glfw](https://github.com/glfw/glfw)                    | Graphic context                            |
| [Glm](https://github.com/g-truc/glm)                    | Mathematics                                |
| [OpenAL](https://github.com/kcat/openal-soft)           | 3D Audio                                   |
| [Valve OpenVR](https://github.com/ValveSoftware/openvr) | Virual Reality SDK, stereoscopic rendering |
| [Bullet3](https://github.com/bulletphysics/bullet3)     | 3D Physics                                 |
| [Dear ImGui](https://github.com/ocornut/imgui)          | Debug menu & GUI                           |
| [Google Test](https://github.com/google/googletest)     | Unit testing                               | 
| [CMake](https://github.com/Kitware/CMake)               | Build system, project generation           | 

This project was intended to be used on Windows, but has a theorical compatibility with Linux and MACOSX.

#### Summary

<ul>
  <li><a href="#Features">Features</a></li>
  <li><a href="#Usage">Usage</a></li>
  <ul>
        <li><a href="#Plugins">Plugins</a></li>
        <li><a href="#Log">Log</a></li>
        <li><a href="#Rendering">Rendering</a></li>
        <li><a href="#Lighting">Lighting</a></li>
        <li><a href="#Virtual Reality">Virtual Reality</a></li>
        <li><a href="#Gizmos">Gizmos</a></li>
  </ul>
  <li><a href="#Gallery">Gallery</a></li>
</ul> 


**Scene example preview**

<p align="center">
  <img src="https://raw.githubusercontent.com/Aredhele/Cardinal-Engine-Archived/main/Press/Cardinal_Capture_ParticleSnow.gif"/>
</p>

<div id="Features"></div>

### Features

The following section will describes core features developped for Cardinal Engine (Archived).

#### General

| Feature                 | Description                                                                               |
|-------------------------|-------------------------------------------------------------------------------------|
| Plugins                 | It allows to have multiple projects at the same time without recompiling the engine |
| Logger                  | Used to log debug, warning or error messages in the engine                          |
| Assertions              | Used to have pre and post-conditions in functions                                   |
| ImGUI support           | Used to draw the GUI of the Engine                                                  |
| Hierarchy and Inspector | Used to inspect and update engine object at runtime using the ImGUI menu            |

#### Rendering (Virtual Reality)

| Feature                 | Description                                                                                                    |
|-------------------------|----------------------------------------------------------------------------------------------------------------|
| OpenVR support          | The render loop supports OpenVR and stereoscopic rendering (2 frames are renderer each frame, one for each eye |
| HTC Vive support        | The engine was tested using an HTC Vive headset and its controllers                                            |
| OpenVR mirror window    | Used to vizualize the stereoscopic rendering on a 2D screen with a side by side eye view                       |

#### Rendering (Post process)

| Feature               | Description                                                    |
|-----------------------|----------------------------------------------------------------|
| Post-processing stack | Used to have multiple post-processing effects at the same time |
| Blur                  | Box blur and Gaussian blur                                     |
| Kernel                | Edge detection, Sharpen                                        |
| Debug                 | Depth buffer, Ligth scattering pass                            |
| Utility               | Mirror, Negative, Identity                                     |
| Depth buffer          | Depth of field, Fog                                            |
| Bloom                 | Fast and experimental bloom                                    |
| God rays              | God rays using a light scattering pass                         |
| AA                    | Implementation of the FXAA                                     |
| Misc                  | Edge detection, Sharpen, Vignette, Sepia tone                  |

#### Rendering (Renderers)

| Feature          | Description                                              |
|------------------|----------------------------------------------------------|
| MeshRenderer     | Used to render all 3D meshes                             |
| LineRenderer     | Used to render debug lines                               |
| TextRenderer     | Used to render 2D texts                                  |
| ParticleRenderer | Optimized rendere for particules using GPU instantiation |

#### Rendering (Lighting)

| Feature             | Description                                              |
|---------------------|----------------------------------------------------------|
| Directional light   | Forward rendering, directional light support             |
| Point light         | Forward rendering, slow point light shader               |

#### Debug

| Feature | Description                                        |
|---------|----------------------------------------------------|
| Gizmos  | Support of debug box, cone, lights, lines and rays |

#### Audio

| Feature              | Description                                                                 |
|----------------------|-----------------------------------------------------------------------------|
| OpenAL Support       | Spatialized 3D audio using OpenAL                                           |
| Audio loading        | The engine is able to load WAV files                                        |
| Listener and sources | The engine can have an audio listener and multiple audio sources (emitters) |

<div id="Usage"></div>

## Usage

This section will enumerate some engine-side implemented features. 

<div id="Plugins"></div>

#### Plugins
Because the engine was designed to create procedural voxel worlds, the code of the project has been quickly split into 
two main parts : Engine and Game. To achieve that, the game part is compiled separately into a plugin that can
be registered later and called by the engine.

The first thing to do is to create a class that will extend the engine plugin interface and implements virtual pure methods.

Code
```cpp
#include "Runtime/Core/Plugin/Plugin.hpp"

class MyPlugin : public cardinal::Plugin
{
public:

    static MyPlugin * s_pPlugin;

public:

    /// \brief Called when the game begins
    void OnPlayStart() final;

    /// \brief Called when the game stops
    void OnPlayStop () final;

    /// \brief Called just before the engine update
    void OnPreUpdate() final;

    /// \brief Called after the engine update
    /// \param dt The elapsed time in seconds
    void OnPostUpdate(float dt) final;

    /// \brief Called when it's time to render the GUI
    void OnGUI() final;
};
```

Then to register your plugin into the engine, you have to implement a function called "OnPluginRegistration()". 
This function will be called after the engine initialization so it is already safe.

Code
```cpp
/// \brief Hook to register user plugin from the static libraries
void OnPluginRegistration()
{
    // Creating our plugin instance
    MyPlugin::s_pPlugin = new MyPlugin();

    // Engine plugin registration
    cardinal::PluginManager::RegisterPlugin(MyPlugin::s_pPlugin);
    
    // After this call, plugin virtual methods will be call automatically
}
```

<div id="Log"></div>

#### Log

At any time in your code you can log debug messages using the engine built-in log 
system. There are 4 log levels : User, Info, Warning and Error.

Code
```cpp
#include "Runtime/Core/Debug/Logger.hpp"

...

cardinal::Logger::LogUser   ("Foo %d", foo); // Write in stdout with [User] tag
cardinal::Logger::LogInfo   ("Foo %d", foo); // Write in stdout with [Info] tag
cardinal::Logger::LogWarning("Foo %d", foo); // Write in stdout with [Warn] tag
cardinal::Logger::LogError  ("Foo %d", foo); // Write in stderr with [Erro] tag
```

Output example

<p align="center">
  <img src="https://raw.githubusercontent.com/Aredhele/Cardinal-Engine-Archived/main/Press/Cardinal_FirstBoot.png"/>
</p>

<div id="Rendering"></div>

#### Rendering

The engine has 4 types of renderers that extend the renderer interface IRenderer :
**MeshRenderer, LineRenderer, TextRenderer and ParticleRenderer**. Each renderer can be allocated by calling the 
appropriate function in the Rendering Engine. Calling one of those functions will return a pointer on an initialized 
renderer ready to be used. The renderer will be **autonomous** and there is nothing more to do, you can send
your vertices, normals and uvs to the renderer and your mesh will be drawn. Let's see the following example :

Code

```cpp
#include "Runtime/Rendering/Renderer/MeshRenderer.hpp"

/// \brief Called when the game begins
void MyPlugin::OnPlayStart()
{
    ...
    
    // Full setup example
    // Loads a bmp texture from a file and registers it as the given ID
    TextureLoader::LoadTexture("MyTextureID", "Resources/Textures/MyTexture.bmp");
    
    // Getting back our texture from its ID
    uint myTextureID = cardinal::TextureManager::GetTextureID("MyTextureID");
    
    // Creating our shader
    cardinal::StandardShader * pShader = new cardinal::StandardShader();
    
    // Setting the texture of the shader
    pShader->SetTexture(myTextureID);
    
    // Allocating the renderer in one call
    MeshRenderer * pMeshRenderer = cardinal::RenderingEngine::AllocateMeshRenderer();
    
    // Initializing by passing all the geometry 
    pMeshRenderer->Initialize(...);
    
    // Finally
    pMeshRenderer->SetShader(pShader);
    
    // To release a renderer after usage
    cardinal::RenderingEngine::ReleaseRenderer(pMeshRenderer);
    
    ...
}
```

**Note that the engine has no managed object. The user should take care to release resources after use to avoid resource leak
(mainly memory leak).**

<div id="Lighting"></div>

#### Lighting

It is possible to illuminate the scene with two kinds of lights : Directional, Point.
Just like renderers, there is only one function to call.

Code

```cpp
#include "Runtime/Rendering/Lighting/Lighting.hh"

/// \brief Called when the game begins
void MyPlugin::OnPlayStart()
{
    // Creating the directional light
    cardinal::LightManager::CreateDirectionalLight();
    
    // Getting an access on it 
    cardinal::DirectionalLight * pDirectional = cardinal::LightManager::GetDirectionalLight();
    
    // The position is debug purpose only
    pDirectional->SetPosition(glm::vec3(128.0f, 128.0f, 300.0f));
    pDirectional->SetDirection(glm::vec3(-0.5f, -0.5f, -0.5f)); 
    
    // For point lights
    cardinal::PointLight * pPoint = cardina::LightManager::AllocatePointLight();
    
    pPoint->SetRange(20);
    pPoint->SetIntensity(1.0f);
    pPoint->SetColor(glm::vec3(1.0f, 1.0f, 1.0f));
    pPoint->SetPosition(glm::vec3(100.0f, 20.0f, 300.0f));
}   
```

Output (Gizmos enabled)

<p align="center">
  <img src="https://raw.githubusercontent.com/Aredhele/Cardinal-Engine-Archived/main/Press/00083_Cardinal_Engine.png"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/Aredhele/Cardinal-Engine-Archived/main/Press/00081_Cardinal_Engine.jpg"/>
</p>

<div id="Virtual Reality"></div>

#### Virtual Reality

That's the nice part. At the end of the project, I was looking for a new idea. I looked at my desk and then I saw my 
HTC vive, and as a VR enthousiast, the idea was clear : How can I look at my own engine in VR! 
After some research, I figured out that Valve published an open source VR SDK called OpenVR that obviously supports the
HTC Vive (but not only). On the tech side, it requires the engine to perform the rendering twice, one complete rendering pass
for each eye since the view matrix et the projection matrix is different between them (not the same position, lenses are different etc.)
Quite heavy for an experimental engine.

To enable stereoscopic rendering just write the following lines at the beginning of your code. HMD stands for "Head Mounted Display".
You also need to have SteamVR installed. Unfortunately, this release doesn't provide a VR Plugin because it requires to much 
maintenance (SDK update etc.) and I don't have the time for it.

Code
```cpp
/// \brief Called when the game begins
void MyPlugin::OnPlayStart()
{
    cardinal::RenderingEngine::InitializeStereoscopicRendering();
    vr::IVRSystem * pHMD = cardinal::RenderingEngine::GetHMD();
    
    // Always check pointers
    ASSERT_NOT_NULL(pHMD);
}
```

The first call will initialize the VR renderer and start SteamVR. It may fail if SteamVR is not
installed or if there is no HMD connected. The engine embeds its own system to mirror frames which are sent to the HMD
before the SteamVR composer passes.

**In Game output VS what it can looks like in real life**
<p align="center">
  <img src="https://raw.githubusercontent.com/Aredhele/Cardinal-Engine-Archived/main/Press/Cardinal_OpenVR_InGame.gif"/>
  <img src="https://raw.githubusercontent.com/Aredhele/Cardinal-Engine-Archived/main/Press/Cardinal_OpenVR_IRL.gif"/>
</p>

<div id="Particle system"></div>

#### Particle system

Cardinal Engine (Archived) has it own basic particle system, **GPU instantiation based**. So it allows you to deal
with a lot of particles at the same time. The emission shape can be either a cone or a plane.
Particle systems can be created through code and modified with the GUI.

Code

```cpp
#include "Runtime/Rendering/Particle/ParticleSystem.hpp"

// Allocating a particle system
cardinal::ParticleSystem * pSystem = cardinal::RenderingEngine::AllocateParticleSystem();

// Initializing the particle system with the following parameters
// Max particle, Emission rate, Life time, Size, Speed, Gravity, Color, Shape
pSystem->Initialize(200000, 5000, 3.0f, 0.5f, 30.0f, glm::vec3(0.0f, 0.0f, -13.0f), glm::vec3(1.0f), new cardinal::Cone(4.0f, 2.0f));

// Setting the position of the particle system
pSystem->SetPosition(glm::vec3(0.0f, 0.0f, 300.0f));
```

Possible output 
<p align="center">
  <img src="https://raw.githubusercontent.com/Aredhele/Cardinal-Engine-Archived/main/Press/00078_Cardinal_Engine.jpg"/>
  <img src="https://raw.githubusercontent.com/Aredhele/Cardinal-Engine-Archived/main/Press/00073_Cardinal_Engine.png"/>
</p>

<div id="Gizmos"></div>

#### Gizmos

At any time, you can request the engine to draw debug information. The engine only supports gizmos for built-in components :
* Box
* Ray
* Line
* Axis
* Grid
* Cone
* Point Light
* Directional Light

To enable or disable a gizmo, just call the EnableGizmo function :

```cpp
#include "Runtime/Rendering/Debug/Debug.hpp"
#include "Runtime/Rendering/Debug/DebugManager.hpp"

/// \brief Called when the game begins
void MyPlugin::OnPlayStart()
{
    // Tells to the engine to draw debug boxes and lines
    cardinal::DebugManager::EnableGizmo(cardinal::DebugManager::EGizmo::Box);
    cardinal::DebugManager::EnableGizmo(cardinal::DebugManager::EGizmo::Line);
}

/// \brief Called after the engine update
/// \param dt The elapsed time in seconds
void MyPlugin::OnPostUpdate(float dt)
{
    // Asks the engine to draw a white box in 0, 0, 0 with a size of 8
    cardinal::debug::DrawBox(glm::vec3(0.0f), 8.0f, 8.0f, glm::vec(1.0f));
    
    // Asks the engine to draw a white line starting at 0, 0, 0 
    // and ending at 8, 8, 8
    cardinal::debug::DrawLine(glm::vec3(0.0f), glm::vec3(8.0f), glm::vec3(1.0f));
}
```

Note : You can also enable or disable gizmos directly from the game with the gizmos window.

<p align="center">
  <img src="https://raw.githubusercontent.com/Aredhele/Cardinal-Engine-Archived/main/Press/00075_Cardinal_Engine.jpg"/>
</p>

<div id="Gallery"></div>

## Gallery

### The Rising Sun, an exotic combination of shaders
<p align="center">
  <img src="https://raw.githubusercontent.com/Aredhele/Cardinal-Engine-Archived/main/Press/00050_Cardinal_Engine.png"/>
</p>

### Orthographic view of a small world
<p align="center">
  <img src="https://raw.githubusercontent.com/Aredhele/Cardinal-Engine-Archived/main/Press/00058_Cardinal_Engine.png"/>
</p>

### Playing arround with particule systems
<p align="center">
  <img src="https://raw.githubusercontent.com/Aredhele/Cardinal-Engine-Archived/main/Press/00065_Cardinal_Engine.png"/>
  <img src="https://raw.githubusercontent.com/Aredhele/Cardinal-Engine-Archived/main/Press/00066_Cardinal_Engine.png"/>
  <img src="https://raw.githubusercontent.com/Aredhele/Cardinal-Engine-Archived/main/Press/00067_Cardinal_Engine.png"/>
</p>

### Wireframe overlay and depth buffer
<p align="center">
  <img src="https://raw.githubusercontent.com/Aredhele/Cardinal-Engine-Archived/main/Press/00079_Cardinal_Engine.jpg"/>
  <img src="https://raw.githubusercontent.com/Aredhele/Cardinal-Engine-Archived/main/Press/00085_Cardinal_Engine.jpg"/>
</p>

### A blurry overview
<p align="center">
  <img src="https://raw.githubusercontent.com/Aredhele/Cardinal-Engine-Archived/main/Press/00086_Cardinal_Engine.jpg"/>
  <img src="https://raw.githubusercontent.com/Aredhele/Cardinal-Engine-Archived/main/Press/00090_Cardinal_Engine.jpg"/>
  <img src="https://raw.githubusercontent.com/Aredhele/Cardinal-Engine-Archived/main/Press/Cardinal_Bloom.png"/>
</p>

### Attempt of a multithreaded endless chunk generation
<p align="center">
  <img src="https://raw.githubusercontent.com/Aredhele/Cardinal-Engine-Archived/main/Press/00009_Cardinal_Engine.png"/>
  <img src="https://raw.githubusercontent.com/Aredhele/Cardinal-Engine-Archived/main/Press/Cardinal_Capture_InfinitePCG.gif"/>
</p>

Thank for reading!