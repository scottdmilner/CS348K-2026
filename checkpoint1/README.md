# Project Check-in 1: Evaluation

## Evaluation 1: Don't Break Anything :)
- Blender ships with a suite of Cycles unit tests that, when ran, test a wide range of capabilities, including the integrator, what we will be modifying. Continuing to pass the unit tests ensures we haven't broken anything.

## Evaluation 2: Feature Parity
- One goal of this is to be able to replicate all of Blender's light passes (diffuse/glossy/transmission/subsurface direct/indirect) as well as lightgroups. Verifying this will check correctness by making sure we haven't lost any paths, as well as allowing the possibility of LPEs being a drop-in replacement for the current system.
- `lpe_testing.py` is a script (based off of the Cycles performance testing script) that can be dropped into Blender's performance testing system that modifies all of the Blender performance testing scenes (good diversity of materials and geometry) outputs EXR files from all of  with the needed AOVs.
- These AOVs should match this list of LPEs:
    - Diffuse Direct: `CD[LO]`
    - Diffuse Indirect: `CD.+[LO]`
    - Glossy Direct: `CD[LO]`
    - Glossy Indirect: `CD.+[LO]`
    - Transmission Direct: `CT[LO]`
    - Transmission Indirect: `CT.+[LO]`
    - Emission: `C[LO]`
    - Lightgroup for light 'name': `C.*<L.'name'>`
- Once LPEs exist in Blender's data model, `lpe_testing.py` will be further modified to also render out this set of LPEs
- Multipart EXR images can be compared with `oiiotool` using this bash script:

```bash
# images to compare
image1=image1.exr
image2=image2.exr
# extract list of layers
channel_list=$(oiiotool -v --info "$image1" 2>&1 | grep "channel list:" | sed 's/.*channel list: //')
layers=$(echo "$channel_list" | tr ',' '\n' | sed -e 's/^[[:space:]]*//' -e 's/[[:space:]]*$//' -e 's/\.[RGBA]$//' -e 's/ViewLayer\.//' | sort -u)

# define layer lookup table:
declare -A lpe_map=(
    ["Combined"]="C.*[LO]"
    ["Combined_BallLight"]="C.*<L.'BallLight'>"
    ["Combined_DomeLight"]="C.*<L.'DomeLight'>"
    ["Diffuse Direct"]="CD[LO]"
    ["Diffuse Indirect"]="CD.+[LO]"
    ...
)

while IFS= read -r layer; do 
    if [[ -n "${lpe_map[$layer]}" ]]; then
        lpe="${lpe_map[$layer]}"
        echo "Diffing Layer ${layer} and LPE: ${lpe}"
    oiiotool -i:ch="ViewLayer.$layer.R,ViewLayer.$layer.G,ViewLayer.$layer.B" "$image1" -i:ch="ViewLayer.$lpe.R,$lpe.G,$lpe.B" "$image2" --diff
    else
        echo "LPE match not found for layer ${layer}"
    fi
done <<< "$layers"
```

## Evaluation 3: New features
- The primary new feature that needs to be evaluated is object tagging. A closeup of the "junkshop" scene will be used for evaluation as it contains fine hair detail. 
- LPE: `C'hair'.*[LO]`, `C.*'hair'.*[LO]`
- There is no ground truth as the feature doesn't currently exist in Cycles, but subtracting off the layer in the compositor and attempting color correction on the layer will be used to test functionality.
- Additionally, a simple blend scene with several mirror balls will be used to test bounce depth captures:
    - `C.{2}[LO]`, `C.{4}[LO]`


## Project Current state
- LPEs are being parsed with a custom patched OSL parsing library and loaded into the integrator. The integrator has been refactored to allow tracing through the automata in the correct order and automata evaluation functions have been added to the integrator. I am currently working on adding LPEs to the list of output layers.
- Code can be found at: https://projects.blender.org/Scott-Milner/blender/compare/aa1f65fd608f05b18284d84ea33e826d83910c9f..97ef809c52401461b0348438dcd2f3debf3c8dcc
