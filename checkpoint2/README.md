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

<img width="952" height="538" alt="Screenshot 2026-05-22 at 11 52 51 PM" src="https://github.com/user-attachments/assets/a4033099-fe52-4c75-ab88-dbda172b72ee" />

Our diffuse pass:

<img width="954" height="538" alt="Screenshot 2026-05-22 at 11 53 19 PM" src="https://github.com/user-attachments/assets/2cdd0129-49e9-4a34-b718-cbd7a3ea367c" />


Diff:

<img width="954" height="537" alt="Screenshot 2026-05-22 at 11 53 59 PM" src="https://github.com/user-attachments/assets/a82383cf-5a12-4146-b81a-c115abbc749b" />


## Evaluation 3: New features
- Still need to be implemented! Currently, light tags and object tags are parsed and passed into the render kernel, but just return black.



# Project Current State
- Support for LPEs added to the Blender DNA/RNA system with Python API support
- LPE output is working, showing that expressions are being evaluated and tracked. The output is not correct yet. 
- Code: https://projects.blender.org/Scott-Milner/blender/compare/main..lpes-stage-1
