# LPEs in Blender Cycles

Scott Milner (sdm222), Stanford CS 348K

Light Path Expressions (LPEs) are an important tool in modern render compositing workflows, allowing generation of artist-defined AOVs for sample-precise object extraction, light, or ray isolation. Smart use of LPEs can significantly reduce the need for re-rendering and improve compositing artist efficiency by reducing the need for matte extraction and enabling relighting workflows. 

LPEs are a tool that is available in production renders such as RenderMan, Karma, Moonray, and Arnold, but is not implemented in Blender Cycles.

This project will supplement (or maybe replace, dependent on performance) Cycles' existing render pass system with a more flexible, artist-controllable LPE-based AOV system.

## Task List
**Stage 0**
- Download and build Blender
- Get all Blender dependencies building locally
- Read through and understand previous two attempted LPE PRs
**Stage 1**: Integrate the OSL code into Cycles, demonstrate basic passes work
- Patch OSL API to access internals of the LPE Parser
- "basic passes work" is successful without dividing out the albedo the way that Blender's current implementation does
- Parse LPEs and load them into the renderer
- Target event/scatter types:
    - **C**: Camera
    - **R**: Reflection
    - **T**: Transmission
    - **L**: Light
    - **O**: Emissive object
    - **D**: Diffuse
    - **G**: Glossy
- Implement RNA/DNA so LPEs can be saved in .blend files
**Stage 2**: Additional event types (goal: feature parity with render layers, plus standard LPE features)
 - Event types
    - **B**: Background/Environment
    - **V**: Volumes
    - **A**: albedo? 
        - alternatively, a prefix tag: `noalbedo;` or similar
        - Is albedo a terminating event, like `B/L/O`? we can transmit through something, then grab its albedo
    - Handle subsurface kernels
    - Object tags
    - Light tags
- Opacity/alpha handling? (particularly for Background event)
- How to handle portals? As its own **P** event type? As a variation of transmission
    - Blender also has a transparent ray type (no refraction)
- Add LPEs (and maybe object groups) to the UI
- Figure out how/if this interacts with shadow catchers

**If I have time/after class is finished**
**Stage 3**: Get OIDN working on LPEs
- Need to be able to reuse OIDN filters across multiple render passes

**Stage 4**: now that we have feature parity, remove the old AOV system
- Create translation so that the old system works fine and lightgroups still work, they are just driven by LPEs under the hood (lightgroups and old AOVs should remain in the UI)
- Whether this makes sense is performance-dependent


## Principles:
- **Don't be wrong**: existing Blender AOVs should be perceptually identical when backed by LPEs instead of the current system
- **Extend existing functionality**: preserve weird Blender quirks like dividing out the albedo, existing workflows shouldn't change
- **Minimal UI modification**: add a single dropdown in the Scene Properties > Passes panel for specifying custom LPEs.
    - Give hover hints with the LPE expression for default AOVs to guide/hint at people who want to do their own
- **(Ideally) Unmodified OSL Source** brecht doesn't want to modify and duplicate the source code from from OSL
- **Performance**: Writing to additional framebuffers will not be free, but this should not tank performance

## Deliverables:
- Renders of Blender test scenes:
    - Demonstrating feature parity with existing render passes
    - Demonstrating features unique to LPEs: object isolation, motion blur/DoF object segmenting, characters behind glass
        - compare to cryptomatte/object ID (previous options)
    - Rendered with arbitrary LPEs
- Performance comparisons for feature parity

## Biggest risks
- It is possible that Stage 1 requires some significant refactoring as Cycles uses rendering kernels that don't execute the render according the sequential model of rendering that LPEs pretend exists


## Resources wanted
- Small amount of compute to run test renders on, including on GPUs. 
