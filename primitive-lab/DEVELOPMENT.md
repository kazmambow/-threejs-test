# Primitive Lab: browser prototype and AE renderer contract

## Target

The final target is an After Effects 3D effect plug-in. Browser-only conveniences must not define the material or geometry data model. Renderer adapters may use Three.js now and C++ / the AE SDK later.

## Shared texture inputs: CH 1–10

These are **not** the distribution/array channels (1–8).

- Each material property stores a stable reference such as `channel:1`, not a copied image.
- Browser provider: each slot references a session asset loaded from a local file. It is shared by all materials. Replacing a slot updates every referencing material.
- Browser lifetime: memory only. Refreshing the page clears all ten inputs and imported assets. No localStorage / IndexedDB persistence.
- AE provider: CH 1–10 must correspond to ten Custom Layer inputs. Resolve the selected AE layer at the requested render time. Keep the same material references, rather than baking the layer into each material.
- Color textures and numeric/normal/displacement data must use the appropriate color space even when a single channel is reused for different purposes.
- Retain immutable session asset IDs while undo history references them; do not overwrite a previous image in place.

## Material source schema

Scalar, color, and boolean values remain ordinary JSON properties. `material.sources[propertyName]` selects `''`, an immutable session asset ID, or `channel:N`. Existing legacy `base`, `rough`, `normal`, and `disp` references are accepted by the resolver.

Numeric property maps use the R channel as a multiplier. IOR maps interpolate the excess above 1. Color maps multiply the selected color after color-space conversion. Normal and Displacement retain their dedicated geometric semantics. Boolean maps use a 0.5 threshold. Names are metadata and do not accept texture maps.

`installMaterialMaps` and `applyMaterial` are browser renderer adapters. Do not encode Three.js Texture/Color objects in material snapshots.

## Environment

Environment source, intensity, rotation, exposure, and background options are scene-level values, not per-material values. HDR/EXR and equirectangular PNG/JPG images are session assets. Browser rendering uses PMREM; the future AE renderer may implement its own lighting integration.

## Local file references

Browser file pickers do not expose an OS file path that can be used to reveal a file in Explorer/Finder. The browser link therefore opens the selected blob/image (HDR/EXR may be downloaded by the browser). Native Reveal in Folder is an AE/native-host feature and is not implemented in this HTML prototype. Never label a browser blob URL as an OS folder path.

## v0.21 validation status

Implementation is prepared on a review branch. DOM/adapter tests cover the new controls, ten slots, replacement, undo restoration, role separation, and environment data. A real RGBELoader decodes the HDR fixture. **Actual browser/GPU rendering has not been verified for v0.21:** macOS sandbox denied Chrome's MachPortRendezvous startup, and the previous computer-use browser tool was unavailable. Keep this branch out of the GitHub Pages production branch until real browser checks pass.

## v0.22: update separation and AE animation

- Material parameter edits update the existing material and its compiled uniform sets. They do not rebuild geometry, distribution groups, or folder instances.
- Shader programs are invalidated for layout/feature changes, not for ordinary value changes. Runtime caches live in WeakMaps, outside JSON scene data.
- Distribution updates reuse geometry/material resources if type, geometry parameters, UV settings, shading mode, and OBJ source are unchanged.
- Parse OBJ mesh geometry once and share it between copies. Dispose geometry/material resources when the shape is replaced or the object is deleted.
- Render when the scene changes, when OrbitControls reports movement, or when damping is active. Idle frames skip GPU rendering; resize/context restoration/visibility and environment changes invalidate the scene.
- AE distribution/Scatter controls should be exposed as keyframeable effect parameters for array channels 1–8. These are separate from texture Custom Layer inputs 1–10.
- At any requested render time, compute placement from evaluated parameters and Seed. Results must not depend on playback order. Caches are an optimization only; an AE implementation must isolate/synchronize caches for concurrent frame rendering.

### Validation

DOM/adapter tests compare v0.21 and v0.22 with real Three.js geometry/material objects and a mocked renderer. Fifty roughness edits change geometry regeneration from 50 to 0. A two-submesh OBJ grid of 125 copies changes initial parses from 125 to 1 and rearrangement parses from 125 to 0. Uniform refresh, cached shader variants, source replacement/removal, folder-shared materials, geometry/UV replacement, idle redraw suppression and camera events are covered. Actual browser/GPU FPS has not been measured; no Chrome process was launched for v0.22 testing.
