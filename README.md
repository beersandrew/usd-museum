# usd-museum

## What

A USD scene with a goal to express the features of USD.

### Why

To attempt a more human-centric specification test, in support of AOUSD.

### How

This scene was built with the help of Claude Code.

## Glossary

| Word | Means |
|---|---|
| **Museum** | The whole thing: `museum.usda`, every wing placed along one route, joined by corridors |
| **Wing** | One big area of USD (Composition, Animation, Materials...). A folder here, with its own `tour.usda` |
| **Hall** | A wing's entrance room: its walls, lamps and signs |
| **Gallery** | One topic inside a wing (in Composition: arcs, strength, edits...). A subfolder of the wing |
| **Exhibit** | One USD asset that demonstrates a topic. It opens on its own: `<wing>/<gallery>/<exhibit>/<exhibit>.usda` |
| **Piece** | One part of an exhibit that shows one rule (in the variants exhibit: default, stronger, fallback, nested) |
| **Reference** | The marks beside a piece that say what must happen: an outline the object must fill, a plinth in the color that must win, a red outline that must stay empty |
| **Forgery** | A copy of an exhibit with one plausible mistake planted. A small override layer in the exhibit's `forgeries/` folder; it renders visibly wrong against the reference |
| **Hold** | The moment in the tour when the camera pauses on an exhibit, which is when a forgery shows |
| **Tour** | `tour.usda`: a camera path through the museum or a wing, with a frame range and a load schedule |
| **Load schedule** | Which galleries (the **carriers**) are loaded at each frame, so a viewer loads only what is near the camera |
| **Feature** | Something USD does that a user would name ("payloads", "variant fallbacks"); see below |

## What USD does: the features

Each feature in plain English, grouped by area, with where the museum shows it. Click an area
to open it.

<!-- features: begin -->

The museum shows 117 of the catalog's 132 features, plus OpenExec (computed values) and UTF-8 names. Each row says what the feature does, then the wings and galleries (`wing/gallery`) where you see it.

<details>
<summary><b>Files, layers and assets</b> (17)</summary>

| Feature | What it does | Where |
|---|---|---|
| usda text format | The human-readable text encoding of a layer: '#usda 1.0' header, layer metadata block, prims, properties, metadata and values. | foundations/files |
| usdc crate binary format | USD's own binary layer encoding: a lazily read, deduplicated, compressed file that is losslessly convertible to and from usda. | foundations/files, foundations/types |
| usdz packages | A read-only, uncompressed, 64-byte-aligned zip archive whose first file is the default layer, holding USD layers plus textures and other assets. | foundations/files, foundations/packages |
| Sublayers and layer stacks | A layer lists other layers as subLayers; the recursive, ordered result (the layer first and strongest) is a layer stack whose opinions combine by strength. | foundations/files, composition/strength, composition/edits, composition/errors, animation/keys, animation/time |
| Layer offsets (time offset and scale) | A sublayer, reference or payload can carry (offset, scale) that remaps the time of everything it brings in; offsets compose down the chain. | foundations/files, animation/time, animation/characters |
| timeCodesPerSecond and framesPerSecond | Layer metadata that maps timecodes to real seconds; layers at a different rate are automatically stretched to the stage's rate when composed. | animation/time |
| startTimeCode and endTimeCode | Root-layer metadata giving the playback range a tool should show first; it does not limit what times can be queried. | animation/time |
| upAxis | Root-layer metadata declaring whether Y or Z is up for all geometry on the stage. | foundations/files |
| metersPerUnit (linear units) | Root-layer metadata giving the length of one scene unit in meters (fallback 0.01, centimeters). | foundations/files, foundations/packages |
| defaultPrim | Layer metadata naming the prim that a reference or payload to the layer gets when it gives no prim path. | foundations/files |
| Asset paths and anchoring | Asset-valued fields (@path@) that are relative are resolved against the layer where they were authored, not the stage root or working directory. | foundations/packages |
| Package-relative asset paths | Paths like pkg.usdz[inner/file.usd] address files inside a package, and relative paths authored inside a package resolve inside it. | foundations/files, foundations/packages |
| Expression variables and variable expressions | Root/session layers define expressionVariables; backquoted expressions in asset paths (sublayers, references, payloads, asset values) and variant selections are evaluated from them. | foundations/model |
| Color space metadata and ColorSpaceAPI | Attributes can carry colorSpace metadata, prims can apply ColorSpaceAPI with colorSpace:name that children inherit, and layers may carry legacy colorConfiguration. | materials/surfaces, materials/color |
| Packaging, localizing and dependency discovery | Finding every file a stage depends on and copying it into a self-contained folder or usdz with asset paths rewritten. | foundations/packages |
| File format plugins and file format arguments | Any file type with a registered SdfFileFormat (mtlx, abc, custom) can be opened, sublayered or referenced as a layer, optionally with :SDF_FORMAT_ARGS: arguments. | materials/surfaces |
| UTF-8 names | Prim and property names can use any Unicode letters, stored as UTF-8 and compared after normalization. | foundations/paths |

</details>

<details>
<summary><b>Composition</b> (25)</summary>

| Feature | What it does | Where |
|---|---|---|
| Strength ordering of arcs (LIVRPS) | When several arcs give a prim opinions, USD ranks them Local, Inherits, Variants, Relocates, References, Payloads, Specializes, and applies that ranking again inside every layer stack an arc brings in. | composition/strength, animation/clips |
| Local opinions and over/def layering | Opinions authored directly at a prim's path anywhere in the current layer stack (root, sublayers, session) beat everything brought in by arcs. | composition/arcs, composition/strength |
| References (external) | A reference composes a prim (and its whole subtree) from another layer onto a prim here, renaming it to the referencing prim's path. | composition/strength, composition/edits, composition/errors, animation/time |
| Internal references | A reference with no asset path (</Path>) copies another prim of the same layer stack, e.g. a prefab bolt reused ten times in one exported file. | composition/arcs, composition/namespace |
| Sub-root references and payloads | An arc may target a prim below the root of the other layer (@set.usda@</Set/Chair>), bringing that prim plus the opinions its ancestors' arcs contribute to it. | foundations/files, composition/arcs |
| Ancestral (inherited-from-parent) arcs | Arcs authored on a prim's ancestors contribute to the prim too, and are weaker than the same kind of arc authored on the prim itself. | composition/strength, instancing/native |
| Repeated arcs, diamonds and cycles | The same layer reached twice through different arcs composes fine (diamond); an arc that leads back to itself is a cycle, reported and skipped. | composition/errors |
| List editing of composition arcs across layers | references, payloads, inherits, specializes and variantSet names are list ops: each layer can prepend, append, delete or replace entries, and the layer stack combines them strongest first. | composition/arcs |
| Inherits and class prims | inherits makes a prim pick up all opinions (properties, metadata, child prims) authored on a source prim, usually an abstract 'class' prim, ranked just below local opinions. | foundations/model, composition/arcs, composition/strength, composition/errors |
| Class overrides through references (implied inherits, local and global classes) | An inherit authored inside a referenced asset is also applied in every referencing layer stack, so overriding /_class_Tree (or a class nested under the model, remapped) in the shot reaches all assets that inherit it. | composition/arcs, composition/strength, instancing/native |
| Specializes | specializes is like inherits, but the base's opinions are globally weakest: anything said about the specialized prim, in any layer stack, beats anything said about the base. | composition/arcs |
| Variant sets and variant selection | A variantSet packages named alternatives on a prim; the selected variant's opinions and child prims compose in, ranked below inherits and above references. | composition/arcs, composition/strength |
| Nested variants | A variant can itself contain variant sets (and selections), so the inner choices available depend on the outer selection. | composition/arcs |
| Where variant selections come from (weaker sites, classes, other variants) | USD finds each variant set's selection by searching the whole prim index strongest-first, including selections that a class, a reference, or another variant authors, re-evaluating as new arcs appear. | composition/arcs |
| Variant fallbacks | When no selection is authored, the application or a plugin can name fallback variants per set (e.g. standin -> render); authored selections always beat fallbacks. | composition/arcs |
| Arcs inside variants | A variant can author references, payloads, inherits, specializes and relocates, so switching a variant swaps whole assets in or out. | composition/arcs, composition/loading |
| Relocates | Layer-level relocates move or rename a prim that came in through an arc (source path -> target path) without editing the source asset; the old path becomes invalid ('salted earth'). | foundations/files, composition/namespace |
| Relocates combined with classes, variants and payloads | A relocated prim keeps the inherits it had at its old place ('spooky inherits'), can be overridden by variants, can come from a payload, and connections pointing at it are remapped. | composition/namespace |
| Payloads | A payload is a reference that can be left unloaded; when loaded it composes exactly like a reference but ranks just below references. | composition/loading |
| Load and unload (working sets, load rules) | A stage can be opened with all, none or some payloads loaded, and prims can be loaded/unloaded later; unloading a prim unloads payloads beneath it. | composition/loading |
| Scenegraph instancing (instanceable prims, prototypes, instance proxies) | A prim marked instanceable that has a composition arc shares one composed prototype with every other prim that composes the same way; overrides below the instance root are ignored. | instancing/native, instancing/crowds |
| Prototype sharing (which instances share a prototype) | Instances share a prototype only if they compose identically below the root (same arcs, same variant selections, same load state and mask); a different selection or an added class arc makes a new prototype. | instancing/native |
| Nested instancing and payloads inside instances | Prototypes can themselves contain instances, and instances can contain payloads; the same override restrictions apply at every level. | instancing/native |
| Path translation of targets across arcs | Relationship targets and attribute connections authored inside an asset are rewritten to the stage paths where things ended up after references, variants and relocates; targets that point outside what was brought in are dropped. | composition/namespace, composition/edits |
| Composition errors and recovery | Bad arcs (missing files, missing target prims, cycles, invalid relocates or targets, conflicting property types) are reported and skipped while the rest of the stage composes. | foundations/paths, composition/errors |

</details>

<details>
<summary><b>Values and time</b> (24)</summary>

| Feature | What it does | Where |
|---|---|---|
| Attribute value types (scalars, vectors, matrices, quaternions, strings, tokens, asset paths, arrays) | Every attribute has exactly one declared value type from a fixed list (bool, int/int64/uint/uint64/uchar, half/float/double, 2-4 vectors, 2-4 matrices, quaternions, string, token, asset, and arrays of all of them), and values must read back exactly as written in both text and binary files. | foundations/types, animation/keys, animation/splines |
| Value roles (point, normal, vector, color, texCoord, frame) | A role name such as color3f or point3f is the same numbers as float3 but tells tools how to interpret them (transform as a position, a normal, a direction, a linear color, a UV). | foundations/types, geometry/meshes |
| Time-valued attributes (timecode and duration) | timecode values are points in time and duration values are lengths of time; when a layer is offset or scaled, timecodes are offset and scaled while durations are only scaled. | foundations/files |
| Default values vs time samples (value resolution order) | An attribute can hold a timeless default and time samples at once; at a frame the strongest layer that has any value wins, and within that layer time samples (then splines) beat the default; asking with no time reads only the default. | animation/keys, animation/splines, stage/imageable, physics/motion |
| Interpolation of time samples (linear, held) and extrapolation | Between two keys USD blends linearly when the type allows it (component-wise for vectors and matrices, slerp for quaternions, element-wise for equal-length arrays) and otherwise holds the earlier key; before the first and after the last key it holds. | animation/keys, animation/splines |
| Attribute variability (uniform vs varying) | 'uniform' marks an attribute as not meant to change over time, but it is only a statement of intent: the core still reads and resolves its time samples. | foundations/model |
| Attribute blocks (None default) | Writing 'attr = None' in a stronger layer hides every weaker default and animation for that attribute, so it reads as having no authored value (the schema fallback, if any, shows through). | animation/blocks |
| Value blocks inside animation (blocked time samples and spline segments) | A None at one time sample (or a spline segment with no value) means 'no value from here until the next key', and unlike an attribute block the schema fallback does not show through. | animation/blocks |
| Animation blocks | Writing 'attr = AnimationBlock' hides weaker time samples and splines but lets a weaker default value through. | animation/blocks |
| Schema fallback values | A schema can give an attribute a value that is reported when nothing is authored (Sphere radius 1, visibility 'inherited'), without writing it to any file. | stage/schemas |
| Value clips (sequenced animation from many files) | 'clips' metadata on a prim pulls time samples for its attributes from a list of clip layers, each active over a stage-time range, as if the samples were authored in place. | foundations/model, animation/clips |
| Clip timing (times mapping, jumps, reversal, offsets) | The clip 'times' list maps stage time to time inside the clip, piecewise linearly, which allows holds, loops, reversal and jumps; layer offsets above the clips shift and scale the whole mapping. | animation/clips |
| Clip sets, template clips and clipSets ordering | A prim can have several named clip sets (each its own dictionary inside 'clips'), ordered by the list-edited 'clipSets' metadata; a set can also name its clips with a template path like 'sim.###.usd' and a start, end and stride. | animation/clips |
| Clip manifests and missing values in clips | A clip manifest lists which attributes the clips animate; when an active clip has no samples for one of them, USD uses the manifest's default or block, or, if asked (interpolateMissingClipValues), interpolates from neighbouring clips. | animation/clips |
| Splines (Ts animation curves) | An attribute can hold a spline of knots (time, value, tangents, per-segment interpolation: held, linear, curve; Bezier or Hermite), evaluated to a scalar value at any time. | foundations/files, animation/splines |
| Spline extrapolation and looping | Before the first and after the last knot a spline can be held, linear, sloped, or loop (repeat, reset, oscillate); 'inner loops' repeat a section in the middle (deprecated in 26.11). | animation/splines |
| Layer-offset timing of values (offset, scale, frame-rate scaling) | When a sublayer or reference carries an offset and scale, or a different timeCodesPerSecond, every time sample, spline knot and clip time it brings in is remapped to stage time. | animation/time |
| List ops (prepend, append, delete, explicit; legacy add/reorder) | List-valued fields (relationship targets, connections, references, inherits, apiSchemas, clipSets, and int/string/token list metadata) are edited per layer with prepend, append and delete, or reset with an explicit list, and the layers' edits combine into one ordered list with no duplicates. | foundations/files, composition/arcs, composition/namespace, composition/edits |
| Dictionaries, customData and customLayerData | Dictionary-valued metadata (customData on prims and properties, customLayerData on layers, and plugin-defined fields) hold nested key/value data that merges key by key across layers rather than strongest-wins. | foundations/model |
| Relationships and target path translation | A relationship is a list-edited set of paths to other prims or properties; when it comes in through a reference, payload, variant or inherit, USD rewrites the authored paths into the stage's namespace. | foundations/paths, composition/namespace |
| Relationship forwarding | A relationship can target another relationship; asking for forwarded targets follows the chain to the final prims or properties, skipping purely forwarding relationships and stopping at cycles. | composition/namespace, lights/shaping |
| Attribute connections | An attribute can carry a list-edited set of connection paths to other attributes (typically an input to an upstream output); USD stores and translates them but computes no dataflow. | foundations/paths, composition/namespace, materials/textures |
| Array edits (sparse edits to array values) | An array-valued attribute can hold an edit instead of a whole array (write value to index, append, prepend, erase, resize, minsize/maxsize) that applies on top of the next weaker array, including per time sample and in clips. | composition/edits |
| OpenExec (computed values) | Controllers compute attribute values, such as a joint's transform, from other attributes, so a rig drives its parts instead of every pose being keyed. | rigging/exec |

</details>

<details>
<summary><b>The scene: prims, schemas, transforms</b> (24)</summary>

| Feature | What it does | Where |
|---|---|---|
| Specifiers: def, over and class | Each prim spec says whether it defines a prim (def), only overrides one if it exists (over), or is an abstract template for inheriting (class); the composed prim takes the strongest def or class, and an over never changes it. | foundations/model, composition/arcs, stage/population |
| Active and inactive prims (non-destructive deletion) | Setting active = false on a prim removes it and everything under it from the composed stage, reversibly, and a stronger layer can set it back to true. | stage/population |
| Default traversal and prim flags | A normal walk of the stage visits only prims that are active, loaded, defined and not abstract; other walks can ask for inactive, unloaded, undefined or class prims explicitly. | foundations/model, stage/population |
| Typed and untyped prims (schema type names) | A prim's typeName (Mesh, Xform, Sphere...) names the schema that gives it built-in properties and behaviour; a prim may have no type, or a type this build doesn't know. | foundations/model, stage/schemas |
| Fallback prim types (fallbackPrimTypes) | Root-layer metadata that says 'if you don't know type X, treat the prim as type Y', tried in order. | stage/schemas |
| Applied API schemas (single-apply) | The apiSchemas list on a prim mixes extra schemas (MaterialBindingAPI, GeomModelAPI...) into its definition, adding built-in properties and answering HasAPI. | foundations/packages, stage/schemas |
| Multiple-apply API schemas (instance names) | Some API schemas (CollectionAPI, AccessibilityAPI, physics limits) can be applied several times to one prim, each with an instance name that becomes part of its property names. | stage/schemas |
| Prim definitions: built-in property metadata and metadata fallbacks | For built-in properties, the schema decides metadata such as variability and allowed tokens, and unauthored prim metadata (active, kind, instanceable) has a defined 'not set' meaning. | stage/schemas |
| Kind and model hierarchy (assembly, group, component, subcomponent) | The kind metadata labels prims as assemblies, groups, components or subcomponents, and the unbroken chain of group-kinds from the root down to components forms the model hierarchy, a cheap table of contents of the scene. | stage/population, stage/imageable |
| Model draw modes (origin, bounds, cards) | model:drawMode on a model asks viewers to replace the model's subtree with a stand-in (its axes, its bounding box, or textured cards); it inherits down the model hierarchy and applies at components unless model:applyDrawMode = false. | instancing/crowds, stage/population, stage/imageable |
| Purpose (default, render, proxy, guide) | Purpose tags imageable subtrees as always drawn (default), final-render only (render), fast-preview only (proxy) or interactive helpers (guide); an imageable prim's authored purpose wins, otherwise it inherits the nearest imageable ancestor's authored purpose. | stage/imageable |
| Visibility and its inheritance | visibility = invisible hides an imageable prim and its whole subtree (pruning); inherited (the fallback) means 'whatever my parent is'; it can be animated. | animation/keys, stage/imageable, stage/meaning |
| Transforms: xformOps and xformOpOrder | A prim's local transform is the ordered list of op attributes named in xformOpOrder (translate, rotate*, scale, orient, transform, with optional suffixes and !invert! pairs), each independently overridable. | foundations/types, animation/keys, geometry/xforms |
| resetXformStack (ignore parent transforms) | Putting !resetXformStack! in xformOpOrder makes the prim ignore every ancestor transform, so its own ops place it directly in world space. | geometry/xforms |
| Extent on boundable prims | extent stores a gprim's local-space bounding box (min, max) for the values authored, so readers don't have to read all the points to bound it. | stage/imageable |
| Computed bounds and model extentsHint | World and local bounds of any subtree are computed from leaf extents and transforms for a chosen set of purposes, skipping invisible prims; models may cache their bounds per purpose in extentsHint. | stage/imageable |
| Primvars and interpolation modes | A primvar is an attribute in the primvars: namespace whose array is spread over the surface by its interpolation: constant (one value), uniform (per face), varying and vertex (per point), faceVarying (per face corner), with elementSize values per element. | stage/schemas, materials/primvars |
| Indexed primvars | A primvar can store unique values once plus an int[] primvars:name:indices array saying which value each element uses; flattening expands it. | materials/primvars |
| Primvar inheritance down namespace | Constant-interpolation primvars authored on an ancestor apply to every imageable descendant that doesn't author its own value. | foundations/paths, instancing/crowds, materials/primvars |
| Collections (includes, excludes, expansion rules) | CollectionAPI:name on a prim defines a named set of prims and properties by including paths, excluding sub-paths, and an expansion rule (explicitOnly, expandPrims, expandPrimsAndProperties); collections can include other collections. | stage/collections |
| Pattern-based collections (membership expressions and mode) | A collection can instead pick its members with a path expression (wildcards, //, {model} predicates, references to other collections, + - &), and the mode attribute or the automatic rule decides whether relationships or the expression are used. | lights/shaping |
| Point instancers | A PointInstancer draws many copies of a few prototype subtrees, choosing a prototype per instance with protoIndices and placing each with positions, orientations and scales. | composition/edits, instancing/points, animation/keys |
| Point instancer masking, ids and motion (inactiveIds, invisibleIds, velocities) | Instances can be hidden by id without editing the arrays (inactiveIds as list-edited metadata, invisibleIds as an animatable attribute), ids track particle identity over time, and velocities/accelerations give correct motion between samples. | instancing/points |
| Property ordering | A composed prim lists its properties in 'dictionary' order (case-insensitive, numbers compared as numbers), not as written, unless the strongest propertyOrder (reorder properties) moves some names to the front. | stage/population |

</details>

<details>
<summary><b>Domains: geometry, shading, lights, skeletons, physics and more</b> (29)</summary>

| Feature | What it does | Where |
|---|---|---|
| Polygon meshes (topology, normals, sidedness, holes) | A Mesh stores points plus per-face vertex counts and indices, with winding orientation, doubleSided, normals, holes and display color. | geometry/meshes |
| Subdivision surfaces (schemes, creases, corners, boundary and face-varying rules) | A Mesh can be a subdivision cage: subdivisionScheme, interpolateBoundary, faceVaryingLinearInterpolation, triangleSubdivisionRule, crease and corner sharpness say how a renderer smooths it. | geometry/meshes |
| Implicit primitives (Sphere, Cube, Cone, Cylinder, Capsule, Plane and the _1 variants) | Analytic shapes defined by a few numbers (radius, height, size, axis) instead of a point list. | geometry/shapes |
| Basis curves and points (hair, grass, particles) | BasisCurves batch many linear or cubic curves (bezier, bspline, catmullRom; nonperiodic, periodic, pinned) into one prim with widths and optional normals; Points store particles with widths. | geometry/curves |
| NURBS patches, NURBS curves and Hermite curves | Rational B-spline surfaces (with trim curves), batched NURBS curves, and Hermite curves that pass through their points with tangents. | geometry/curves |
| Tetrahedral meshes | A TetMesh is a set of tetrahedra (4 indices each) plus optional surface triangles, for volumetric simulation. | geometry/meshes |
| Geometry subsets and families (incl. per-face materials) | GeomSubset child prims name sets of faces, points, edges, segments or tetrahedra of their parent; familyName and familyType group them, and the materialBind family gives per-face materials. | materials/binding |
| Motion: velocities, accelerations and motion blur | Point-based prims can carry velocities and accelerations so positions can be extrapolated between samples; MotionAPI's motion:blurScale and nonlinearSampleCount control blur. | animation/motion |
| Cameras (projection, lens and filmback units, clipping, depth of field, shutter, exposure, stereo) | A Camera prim describes the frustum, focus, shutter and exposure, looking down -Z with +Y up in its own space. | cameras/lenses, cameras/render |
| Materials, shaders, node graphs, connections and interface inputs | Shading networks are prims (Material, NodeGraph, Shader) whose inputs and outputs connect, with public interface inputs on containers that drive inner nodes. | materials/textures |
| Shader identity and render contexts (info:id, sourceAsset, sourceCode, per-renderer outputs) | A Shader says what it is by registry id or by a source file or inline code per source type, and a Material can carry a universal outputs:surface plus renderer-specific outputs like outputs:ri:surface or outputs:mtlx:surface. | materials/surfaces |
| Material binding: direct, inherited, binding strength and material purpose | A prim binds a Material with material:binding (optionally :preview or :full); bindings inherit down namespace, deeper wins unless an ancestor says strongerThanDescendants, and the purpose-specific binding beats the all-purpose one. | foundations/paths, stage/schemas, materials/binding |
| Collection-based material binding | material:binding:collection:<name> targets a collection and a material, so one relationship assigns a look to many prims. | stage/population, stage/schemas, stage/collections, materials/binding |
| UsdPreviewSurface, UsdUVTexture, primvar readers and UsdTransform2d | A small, fixed physically-based material and texture node set that every USD consumer is expected to understand. | materials/surfaces |
| MaterialX (.mtlx as a USD layer, ND_ nodes) | A .mtlx document can be referenced like a USD layer and appears as Materials, NodeGraphs and ND_* shaders; MaterialX networks can also be authored directly in USD. | materials/surfaces |
| Light types and common light controls (units, exposure, normalize, color temperature, shaping, shadows, light lists) | Sphere, Rect, Disk, Cylinder and Distant lights (and PluginLight) share LightAPI controls: intensity × 2^exposure in nits, normalize, color temperature, ShapingAPI cones and IES, ShadowAPI, and a LightListAPI cache for finding lights. | lights/emission, lights/shaping |
| Dome lights (environment maps, texture formats, pole axis, DomeLight_1) | A DomeLight lights the scene from an environment texture at infinity; texture:format and (on DomeLight_1) poleAxis say how the image wraps the sphere. | lights/environment |
| Light linking, shadow linking, light filters and filter linking | Each light has lightLink and shadowLink collections (includeRoot true by default) saying what it lights and what casts its shadows; light:filters attach LightFilters, which have their own filterLink collection. | lights/shaping |
| Skeletons and joint animation | A Skeleton lists joints as path tokens with bind and rest transforms; a SkelAnimation supplies vectorized translations, quaternion rotations and scales over time; skel:skeleton and skel:animationSource bind them under a SkelRoot. | instancing/crowds, animation/characters |
| Skinning (joint influences, geomBindTransform, linear and dual-quaternion) | SkelBindingAPI primvars skel:jointIndices and skel:jointWeights (with elementSize) and skel:geomBindTransform tell how points follow joints, by classicLinear or dualQuaternion skinning. | animation/characters |
| Blend shapes and inbetweens | BlendShape prims hold point offsets (optionally sparse by pointIndices, with normalOffsets) and weighted inbetween shapes; the SkelAnimation's blendShapeWeights drive them by name. | animation/characters |
| Physics: scenes, rigid bodies, colliders, mass and physics materials | PhysicsScene sets gravity; PhysicsRigidBodyAPI, PhysicsCollisionAPI (+MeshCollisionAPI approximation), PhysicsMassAPI and PhysicsMaterialAPI describe rigid-body simulation, in units set by metersPerUnit and kilogramsPerUnit. | physics/motion |
| Physics joints, limits, drives and articulations | Joint prims connect body0 and body1 at local frames (localPos/localRot); revolute, prismatic, spherical, distance and fixed joints take multiple-apply LimitAPI and DriveAPI per axis; ArticulationRootAPI marks reduced-coordinate chains. | physics/motion |
| Render settings, products, vars and passes | RenderSettings names a camera, resolution and products; RenderProducts list ordered RenderVars (AOVs); RenderPass ties a settings prim to pass-specific collections (renderVisibility, cameraVisibility, prune, matte) and input passes. | cameras/render |
| Spatial audio, asset previews and authorship (UsdMedia) | SpatialAudio places a sound in the scene (spatial or ambient) with playback timing; AssetPreviewsAPI stores thumbnails in assetInfo; AuthorshipAPI (new in 26.11) records who authored what. | animation/characters |
| Volumes and particle fields (OpenVDB, Field3D, Gaussian splats) | A Volume prim names its fields through field:<name> relationships to OpenVDBAsset/Field3DAsset prims (file, fieldName, index); ParticleField prims (26.03+) describe particle or Gaussian-splat radiance fields. | geometry/volumes |
| Semantic labels (UsdSemantics) | Multiple-apply SemanticsLabelsAPI:<taxonomy> puts token lists of labels on prims; descendants accumulate their ancestors' labels. | stage/meaning |
| usdz for AR: anchoring, preliminary schemas and the web/AR-compliant profile | Apple's Preliminary_ schemas (AnchoringAPI, physics, behaviors, text) ride along as API schemas USD doesn't know, and usdchecker --arkit / usdzip --arkitAsset enforce a restricted profile: self-contained package, allowed file types, UsdPreviewSurface only, no nested gprims. | foundations/packages |
| Level of detail (UsdLod), new in 26.x | LODRootAPI marks a prim whose children are alternative representations; heuristic prims (distance, screen size) per domain (imaging, physics, audio) tell a renderer which to show; LODOverrideAPI forces a choice. | stage/meaning |

</details>

Not in the museum yet (15): Layer documentation, comment and customLayerData; Default resolver, search paths and resolver contexts; Session layer; Edit targets; Layer muting; Saving and exporting layers; Flattening a stage; Flattening a layer stack; Layer identity, anonymous layers and reload; Population masks; Namespace editing (move, rename, delete composed prims); Dynamic payloads (file format arguments from prim metadata); Asset info (assetInfo); Prim child ordering; Geometry and volume lights (MeshLightAPI, VolumeLightAPI, materialSyncMode).

<!-- features: end -->

## The whole museum

| File | What it is |
|---|---|
| [museum.usda](museum.usda) | The museum as one assembly: ten wings, each referenced from `./<wing>/<wing>.usda` and placed and timed along one route, plus the corridors between them |
| [tour.usda](tour.usda) | The guided tour, 16,665 frames (11.6 minutes): sublayers `museum.usda` and adds the camera, the load schedule and the doors closed between wings |
| [tour_cam.usdc](tour_cam.usdc) | The tour camera's path, referenced by `tour.usda` (binary: about 16,000 samples) |

The whole tour runs with OpenExec off, so it leaves out the Rigging wing, which is its own
tour (`rigging/tour.usda`).

## Wings

Eleven wings, in tour order:

| Wing | Galleries | Exhibits | Forgeries |
|---|---|---|---|
| [foundations](foundations/foundations.usda) | files, model, packages, paths, types | 8 | 53 |
| [composition](composition/composition.usda) | arcs, loading, strength, namespace, edits, errors | 9 | 97 |
| [instancing](instancing/instancing.usda) | native, points, crowds | 3 | 23 |
| [animation](animation/animation.usda) | keys, time, splines, clips, blocks, characters, motion | 7 | 72 |
| [rigging](rigging/rigging.usda) | exec | 2 | 9 |
| [stage](stage/stage.usda) | population, schemas, collections, imageable, meaning | 10 | 53 |
| [geometry](geometry/geometry.usda) | shapes, meshes, curves, volumes, xforms | 7 | 36 |
| [materials](materials/materials.usda) | surfaces, textures, binding, primvars, color | 6 | 37 |
| [lights](lights/lights.usda) | emission, shaping, environment | 4 | 11 |
| [cameras](cameras/cameras.usda) | lenses, render | 4 | 22 |
| [physics](physics/physics.usda) | motion | 2 | 7 |

62 exhibits and 420 forgeries. 410 forgeries are override layers; 10 are edited copies
because overrides can't express them (relocates, MaterialX and a few others; each says why).

Rigging is the one wing that runs with OpenExec on (its `tour.usda` says `exec = 1` in
`customLayerData`). It holds OpenExec's two exhibits, split out of Animation on Nov 9, so
Animation runs with exec off.

## Viewing

Open a file in OpenUSD's usdview and press Play. Tours play through their camera. Put
`config/variantFallbacks` on the plugin path first, for the one piece that needs it (see
Gotchas):

```sh
export PXR_PLUGINPATH_NAME=$PWD/config/variantFallbacks
usdview tour.usda --camera /museum/tour/tour_cam                  # the whole museum
usdview composition/tour.usda --camera /composition/tour/tour_cam # one wing
usdview composition/arcs/arcs/arcs.usda                           # one exhibit
usdview composition/arcs/arcs/forgeries/noinherit.usda            # the same exhibit, one planted mistake
```

The Rigging wing needs OpenExec, which usdview reads from the environment:

```sh
USDIMAGINGGL_ENGINE_ENABLE_EXEC_SCENE_INDEX=1 usdview rigging/tour.usda --camera /rigging/tour/tour_cam
```

Stock usdview loads every payload; the whole museum is light enough (about 12,000 prims).
[usd-tools](https://github.com/beersandrew/usd-tools) `view/view.sh FILE` adds conveniences:
the tour camera, OpenExec and the workarounds in Gotchas set from the file, and the load
schedule in `tour.usda` applied as the camera moves.

## Gotchas

- **Variant fallbacks need a plugin** (`composition/arcs/variants`, the "fallback" piece):
  global variant fallbacks come from a plugin, never from a layer. The museum ships it:
  [config/variantFallbacks/plugInfo.json](config/variantFallbacks/plugInfo.json) (`wrap` falls back
  to `checks`). Without it on `PXR_PLUGINPATH_NAME`, the piece shows its `nofallback` forgery.
- **Skinned figures on Metal** (`animation/characters/skinning`, `instancing/crowds/crowds`):
  OpenUSD 26.11 on Metal drops GPU-skinned meshes over many frames. Set
  `USDSKELIMAGING_FORCE_CPU_COMPUTE=1` (usd-tools' `view.sh` does) to use the CPU path.
- **OpenExec is a switch in the environment, not in the file.** The Rigging wing only moves with
  `USDIMAGINGGL_ENGINE_ENABLE_EXEC_SCENE_INDEX=1`; the whole tour leaves Rigging out for that reason.
- **At most 16 lights.** Storm draws only the first 16 lights by path. Each hall's lamps are lit only
  while the camera is near, so no frame has more than 16 on.
- **Five exhibits fail `usdchecker` on purpose.** `usdchecker` passes on every other exhibit; those five
  break USD to show what happens, and each one's `customData["museum"]["expected_errors"]` says why.
  The wings and tours that hold them fail with exactly those errors.
- **The files are generated.** They are built by [usd-tour](../usd-tour); a hand edit here is lost at the next export.
