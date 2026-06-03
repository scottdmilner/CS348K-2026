# Final Report

Scott Milner (sdm222)


In this project, we implement Light Path Expressions in Blender Cycles and demonstrate their application in object matte extraction, a novel feature for Blender. 

# Background

Light Path Expressions (LPEs) are a regex-like syntax for describing a series of events in path tracing. We can use them to capture arbitrary subsets of paths into Arbitrary Output Variables (AOVs) for future use in the image processing pipeline. 


Examples from our implementation:


**Beauty pass:** the ground truth render

<img width="2000" height="1000" alt="beauty" src="https://github.com/user-attachments/assets/14e920d4-ee01-4bec-9c25-22e465a44788" />

**Example 1 `CDL`:** Camera > Diffuse bounce > termination at Light

<img width="2000" height="1000" alt="CDL" src="https://github.com/user-attachments/assets/38e3ec36-3c7f-4914-8d49-dfc4c8a0e794" />

**Example 2 `CD.L`:** Camera > Diffuse bounce > Any bounce > termination at Light

<img width="2000" height="1000" alt="CD L" src="https://github.com/user-attachments/assets/3ce08d0c-c713-4e01-9839-e6b0bdae4096" />

**Example 3 `CD..L`:** Camera > Diffuse bounce > Any 2 bounces > termination at Light

<img width="2000" height="1000" alt="CD L" src="https://github.com/user-attachments/assets/53e76833-8c74-450b-b278-f3fae00b3dc9" />

**Example 4 `CD...L`:** Camera > Diffuse bounce > Any 3 bounces > termination at Light

<img width="2000" height="1000" alt="CD L" src="https://github.com/user-attachments/assets/4dcb3a7c-a453-4b44-aac5-7c33b0aebd6b" />

**Example 5 `C[SG].*L`:** Camera > Glossy or Singular bounce > 0 or more bounces > termination at Light

<img width="2000" height="1000" alt="CSG pLtif" src="https://github.com/user-attachments/assets/ff89b983-49b6-4ed3-bb80-1fe89bb90c91" />

**Example 6 `C'character'.*L`:** Camera > Any bounce tagged with `character` > 0 or more bounces > termination at Light

<img width="2000" height="1000" alt="character" src="https://github.com/user-attachments/assets/202c342e-ffe7-4cb3-af95-2f1489a30103" />


# Our LPE Implementation

This implementation of LPEs uses the Open Shading Language LPE parser to construct a finite-state automaton that tracks whether a particular bounce should be accumulated into an LPE or not. All LPE expressions are flattened into a single automaton for optimal performance. The OSL automaton is extracted, sent to the kernel, and manipulated during render by a re-implementation of the OSL accumulation code in the Cycles integrator. 

The resulting LPEs are made available to the artist via the standard AOV accessibility methods, either through a dropdown in the Render Image view or as a socket on the Render Layer node in compositing.

# Object Matte Extraction

Object matte extraction is the process of generating an AOV that can be used as a screen-space mask for manipulating a render at compositing time. In live-action compositing, this is traditionally done with methods like green screen chroma keying or rotoscoping. In animation and VFX, there are several ways of generating object matte AOVs, each with their own benefits and limitations:


| Method           | Space Complexity                              | Time Complexity | Precision              | Dynamic | In Cycles |
| ---------------- | --------------------------------------------- | --------------- | ---------------------- | ------- | --------- |
| Object ID        | $O(1)$ (1x float)                             | $\sim O(1)$     | Binary only            | ❌       | ✅         |
| Cryptomatte      | $O(1)$ (3x float3 + metadata)                 | $\sim O(1)$     | Alpha only             | ✅       | ✅         |
| Render Layers    | $O(N)$ (Nx float3)                            | $O(N)$          | Color weight           | ❌       | ✅         |
| LPEs             | $O(N)$ (Nx float3)                            | $\sim O(1)$     | Color weight           | ❌       | ❌         |
| Deep Compositing | $O(1)$ (# Z-depth buckets-per-pixel x float3) | $\sim O(1)$     | Color weight + Z-depth | ✅       | ❌         |

Notably, in order to perfectly remove or modify an object in a scene, especially if the scene contains motion blur, depth-of-field, or very fine subpixel detail (such as hair), we need color weight precision, alpha or binary precision is not sufficient. Cycles does not currently provide color weight object mattes in $O(1)$ time, a gap that can be solved with LPEs.

## Example (Qualitative analysis)

Here is a simple scene with a monkey that is moving very fast:

<img width="500" height="500" alt="monkey" src="https://github.com/user-attachments/assets/432ec93b-4c8f-484c-8493-3a3c247a2c49" />


Methods that are binary or alpha-only fail catastrophically at extracting a clean matte. The compositing artist working downstream will need to spend time feathering and cleaning the mask to be able to use it without causing a noticeable "halo" effect, where their changes either spill over into the background or do not actually reach the edges of the foreground.

<img width="500" height="500" alt="monkey_id" src="https://github.com/user-attachments/assets/074e4515-3e8e-4b39-bc2f-2535994dca27" />

*Object ID*

<img width="500" height="500" alt="monkey_crypto" src="https://github.com/user-attachments/assets/4b60b95d-ad9d-4aa2-a779-382a7a58c6fb" />

*Cryptomatte*

In contrast, the LPE `C'monkey'.*L` captures the color weight of the monkey only, providing a clean mask.

<img width="500" height="500" alt="monkey_lpe" src="https://github.com/user-attachments/assets/a1279cbd-9938-4652-8dc6-ce7e291ec380" />

We can also use this for more complicated extractions, such as the hair on this character (note the sub-pixel detail being cleanly separated).

<img width="172" height="156" alt="ezgif-6accfdcd7faaad2d" src="https://github.com/user-attachments/assets/ec92fedc-ea6b-4dc1-a550-62873ffe569f" />


# Performance

Our code introduces a notable render time overhead to the renderer, but this quickly amortizes out when compared with the $O(N)$ performance of using Render Layers for object ID extractions. 

Testing performed on Blender Open Data "Junkshop" test scene, M1 Max, CPU rendering, median 3 of 5 trials.


| Build                         | \# of Mattes | Time (s) |
| ----------------------------- | ------------ | -------- |
| v5.1.1 stable (Render Layers) | 1            | 3.74s    |
| v5.2.0 ours (LPEs)            | 1            | 4.29s    |
| v5.1.1 stable (Render Layers) | 3            | 11.22s   |
| v5.2.0 ours (LPEs)            | 3            | 4.40s    |

When extracting 3 object mattes, LPEs provide a speedup of **~2.5x**.

# Future work

In order to get this code production-ready for submission to the Blender project, the following remains to be implemented:

- other bounce types (transmission, subsurface, volume)
- light tagging (similar to object tagging)
- kernel-level toggle (don't incur the extra render time overhead if the user doesn't request LPEs)

Regarding denoising, I had initially hoped to implement a denoising setup that batch-denoised LPEs in the way that the RenderMan denoiser can, but this is currently beyond the abilities of Intel OIDN, Blender's final-frame denoiser. As you can see in the image diff below (multiplied by 5x for visibility), after denoising, LPE object mattes are still very close, but no longer provide exact sample-precise matte extraction.

<img width="538" height="230" alt="Screenshot 2026-06-03 at 7 44 28 AM" src="https://github.com/user-attachments/assets/844f419a-12dd-421e-b995-af84814f96cd" />


# Code

Source code is available [here](https://projects.blender.org/Scott-Milner/blender/compare/main..lpes-project-code)

# Sources

- [OSL LPE documentation](https://github.com/AcademySoftwareFoundation/OpenShadingLanguage/wiki/OSL-Light-Path-Expressions#event-scattering-type)
- [RenderMan LPE documentation](https://rmanwiki-27.pixar.com/space/REN27/542234535/Light+Path+Expressions)
- There have been 2 previous attempts at adding LPEs to Cycles. Neither implementation works entirely correctly, and neither was ever added to Blender, but both were very useful when getting started and in finding my way around a large, unfamiliar codebase.
    - [Kevin Dietrich, 2022](https://archive.blender.org/developer/D15972) 
    - [Quentin Sanhes, 2026](https://projects.blender.org/blender/blender/pulls/154773/files#diff-a2aac641f4c991b131331333dacc16b1d55376af)
