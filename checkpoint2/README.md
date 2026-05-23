> For this checkpoint, we are expecting that you have the evaluation you submitted for the first checkpoint, but with some meaningful intermediate results. This could be a comparison of your method against a baseline, a set of generation outputs from your current system for all your test prompts, etc. It would also be helpful to present these results in the form of graphs, tables or images (depending on your evaluation).
> 
> Using your current results, you should then be able to tell us what you have done or answered successfully relative to your project goals or questions, and what is still not up to par or unanswered (and how you plan to address that), according to your evaluation framework.


> In general, this is the week where you show us a couple of things:
> 1. You've solidified your evaluation plan from the first checkpoint in response to CA feedback, or to new findings of your own.  In response you now have a good picture of what you are going to show: a demo? a set of images? a graph? a table?  The best way to articulate that is to stub it out, and leave placeholders for data that doesn't exist.
> 2. You've taken steps to fill in the stubbed out evaluation with real data. That might mean you've run your baselines and have the final images or numbers?  Or that might mean the initial version of your algorithm is complete and you have a preliminary result.
> 3. Then, the rest of your project is filling in the evaluation template that you've put forth.  e.g., doing the work to fill in missing numbers/images? making the numbers go "up", making the images better, etc.



# How do I match up to last check-in's evaluation metric?

## Evaluation 1: Don't Break Anything :)
- The code currently passes all of the Cycles unit tests!
```
The following tests passed:
        cycles
        cycles_bake_cpu
        cycles_attributes_cpu
        cycles_bsdf_cpu
        cycles_camera_cpu
        cycles_colorspace_cpu
        cycles_denoise_cpu
        cycles_denoise_animation_cpu
        cycles_displacement_cpu
        cycles_hair_cpu
        cycles_image_colorspace_cpu
        cycles_image_data_types_cpu
        cycles_image_mapping_cpu
        cycles_image_texture_limit_cpu
        cycles_instancing_cpu
        cycles_integrator_cpu
        cycles_light_cpu
        cycles_light_group_cpu
        cycles_light_linking_cpu
        cycles_mesh_cpu
        cycles_motion_blur_cpu
        cycles_node_inlining_cpu
        cycles_openvdb_cpu
        cycles_pointcloud_cpu
        cycles_principled_bsdf_cpu
        cycles_raycast_cpu
        cycles_render_layer_cpu
        cycles_reports_cpu
        cycles_shader_cpu
        cycles_shadow_cpu
        cycles_shadow_catcher_cpu
        cycles_sss_cpu
        cycles_texture_cpu
        cycles_volume_cpu
        cycles_osl_cpu
        cycles_image_mipmap_cpu
        cycles_osl_compile_test

100% tests passed, 0 tests failed out of 37

Total Test time (real) = 268.86 sec
```

## Evaluation 2: Feature Parity
- The code currently does not meet feature parity for Blender's light passes

I get outputs for direct and indirect passes, but they do not match up with Blender's passes. Currently not all rays are being captured, and some diffuse rays are captured as glossy and vice-versa.

Ground-truth diffuse pass:
![[Screenshot 2026-05-22 at 11.52.51 PM.png]]

Our diffuse pass:
![[Screenshot 2026-05-22 at 11.53.19 PM.png]]

Diff:

![[Screenshot 2026-05-22 at 11.53.59 PM.png]]

## Evaluation 3: New features
- Still need to be implemented! Currently, light tags and object tags are parsed and passed into the render kernel, but just return black.



# Project Current State
- Support for LPEs added to the Blender DNA/RNA system with Python API support
- LPE output is working, showing that expressions are being evaluated and tracked. The output is not correct yet. 
- Code: https://projects.blender.org/Scott-Milner/blender/compare/main..lpes-stage-1
