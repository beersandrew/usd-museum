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
| **Reference** | The marks beside a piece that say what must happen: an outline the object must fill, a base in the color that must win, a red outline that must stay empty |
| **Forgery** | A copy of an exhibit with one plausible mistake planted. A small override layer in the exhibit's `forgeries/` folder; it renders visibly wrong against the reference |
| **Hold** | The moment in the tour when the camera pauses on an exhibit, which is when a forgery shows |
| **Tour** | `tour.usda`: a camera path through the museum or a wing, with a frame range and a load schedule |
| **Load schedule** | Which galleries (the **carriers**) are loaded at each frame, so a viewer loads only what is near the camera |
| **Feature** | Something USD does that a user would name ("payloads", "variant fallbacks"); see below |

## Features

<!-- features: begin -->

The museum shows 153 features of USD, in five areas; the 25 it doesn't show yet are at the end. Each row says what the feature does, then the wings and galleries (`wing/gallery`) where you see it.

### 1. Files, layers and assets

| # | Feature | What it does | Where |
|---|---|---|---|
| 1 | usda text layers | A .usda file is a layer written as readable text: the same scene description as any other format. | foundations/files |
| 2 | usda grammar | The text format's grammar: comments, @@@ asset paths, string escapes, exponent numbers and the legacy add list op. | foundations/files |
| 3 | usdc crate files | A .usdc file is a layer in the compact binary crate format: the same scene, faster to read. | foundations/files, foundations/types |
| 4 | Crate versions | Each crate file says its version (0.8 to 0.13), and each version adds what it can store: layer offsets, timecodes, path expressions, relocates, splines. | foundations/files |
| 5 | usdz packages | A read-only, uncompressed, 64-byte-aligned zip archive whose first file is the default layer, holding USD layers plus textures and other assets. | foundations/files, foundations/packages |
| 6 | Sublayers and layer stacks | A layer lists other layers as subLayers; the recursive, ordered result (the layer first and strongest) is a layer stack whose opinions combine by strength. | foundations/files, composition/strength, composition/errors, animation/keys, animation/time |
| 7 | Layer offsets (time offset and scale) | A sublayer, reference or payload can carry (offset, scale) that remaps the time of everything it brings in; offsets compose down the chain. | foundations/files, animation/time, animation/characters |
| 8 | timeCodesPerSecond and framesPerSecond | Layer metadata that maps timecodes to real seconds; layers at a different rate are automatically stretched to the stage's rate when composed. | animation/time |
| 9 | upAxis | Root-layer metadata declaring whether Y or Z is up for all geometry on the stage. | foundations/files |
| 10 | metersPerUnit (linear units) | Root-layer metadata giving the length of one scene unit in meters (fallback 0.01, centimeters). | foundations/files, foundations/packages |
| 11 | defaultPrim | Layer metadata naming the prim that a reference or payload to the layer gets when it gives no prim path. | foundations/files |
| 12 | Asset paths and anchoring | Asset-valued fields (@path@) that are relative are resolved against the layer where they were authored, not the stage root or working directory. | foundations/packages |
| 13 | Package-relative asset paths | Paths like pkg.usdz[inner/file.usd] address files inside a package, and relative paths authored inside a package resolve inside it. | foundations/files, foundations/packages |
| 14 | Expression variables and variable expressions | Root/session layers define expressionVariables; backquoted expressions in asset paths (sublayers, references, payloads, asset values) and variant selections are evaluated from them. | foundations/model |
| 15 | Texture color spaces | A texture reader's sourceColorSpace says whether a file's values are sRGB-encoded or raw data. | materials/surfaces |
| 16 | Color spaces of values | A color's space is its attribute's colorSpace metadata if authored, else the nearest ColorSpaceAPI up the namespace, else linear Rec.709; colorSpace:name without the API applied is ignored. | materials/surfaces, materials/color |
| 17 | Packaging, localizing and dependency discovery | Finding every file a stage depends on and copying it into a self-contained folder or usdz with asset paths rewritten. | foundations/packages |
| 18 | File format plugins and file format arguments | Any file type with a registered SdfFileFormat (mtlx, abc, custom) can be opened, sublayered or referenced as a layer, optionally with :SDF_FORMAT_ARGS: arguments. | materials/surfaces |
| 19 | Paths: relative and namespaced | A path written relative (../../Looks/Gold, ../.inputs:tint) resolves from the prim or property that authored it; a namespaced name (paint:base) is one name, colons and all. | foundations/paths |
| 20 | UTF-8 names | Prim and property names can use any Unicode letters, stored as UTF-8 and compared after normalization. | foundations/paths |

### 2. Composition

| # | Feature | What it does | Where |
|---|---|---|---|
| 21 | Strength ordering of arcs (LIVRPS) | When several arcs give a prim opinions, USD ranks them Local, Inherits, Variants, Relocates, References, Payloads, Specializes, and applies that ranking again inside every layer stack an arc brings in. | composition/strength, animation/clips |
| 22 | Local opinions and over/def layering | Opinions authored directly at a prim's path anywhere in the current layer stack (root, sublayers, session) beat everything brought in by arcs. | composition/arcs, composition/strength |
| 23 | References (external) | A reference composes a prim (and its whole subtree) from another layer onto a prim here, renaming it to the referencing prim's path. | composition/strength, composition/errors, animation/time |
| 24 | Internal references | A reference with no asset path (</Path>) copies another prim of the same layer stack, e.g. a prefab bolt reused ten times in one exported file. | composition/arcs, composition/namespace |
| 25 | Sub-root references and payloads | An arc may target a prim below the root of the other layer (@set.usda@</Set/Chair>), bringing that prim plus the opinions its ancestors' arcs contribute to it. | foundations/files, composition/arcs |
| 26 | Ancestral (inherited-from-parent) arcs | Arcs authored on a prim's ancestors contribute to the prim too, and are weaker than the same kind of arc authored on the prim itself. | composition/strength |
| 27 | Diamonds (repeated arcs) | When two arcs bring in the same layer (a diamond), it is composed once, at its strongest place. | composition/errors |
| 28 | Cycles | An arc that would bring in something it is already inside (a reference, inherit or sublayer cycle) is reported and skipped; the rest still composes. | composition/errors |
| 29 | List editing of composition arcs across layers | references, payloads, inherits, specializes and variantSet names are list ops: each layer can prepend, append, delete or replace entries, and the layer stack combines them strongest first. | composition/arcs, composition/strength |
| 30 | Inherits and class prims | inherits makes a prim pick up all opinions (properties, metadata, child prims) authored on a source prim, usually an abstract 'class' prim, ranked just below local opinions. | foundations/model, composition/arcs, composition/errors |
| 31 | Class overrides through references (implied inherits, local and global classes) | An inherit authored inside a referenced asset is also applied in every referencing layer stack, so overriding /_class_Tree (or a class nested under the model, remapped) in the shot reaches all assets that inherit it. | composition/arcs, composition/strength |
| 32 | Specializes | specializes is like inherits, but the base's opinions are globally weakest: anything said about the specialized prim, in any layer stack, beats anything said about the base. | composition/arcs |
| 33 | Variant sets and variant selection | A variantSet packages named alternatives on a prim; the selected variant's opinions and child prims compose in, ranked below inherits and above references. | composition/arcs |
| 34 | Nested variants | A variant can itself contain variant sets (and selections), so the inner choices available depend on the outer selection. | composition/arcs |
| 35 | Where variant selections come from (weaker sites, classes, other variants) | USD finds each variant set's selection by searching the whole prim index strongest-first, including selections that a class, a reference, or another variant authors, re-evaluating as new arcs appear. | composition/arcs |
| 36 | Variant fallbacks | When no selection is authored, the application or a plugin can name fallback variants per set (e.g. standin -> render); authored selections always beat fallbacks. | composition/arcs |
| 37 | Arcs inside variants | A variant can author references, payloads, inherits, specializes and relocates, so switching a variant swaps whole assets in or out. | composition/arcs, composition/loading |
| 38 | Relocates | Layer-level relocates move or rename a prim that came in through an arc (source path -> target path) without editing the source asset; the old path becomes invalid ('salted earth'). | foundations/files, composition/namespace |
| 39 | Payloads | A payload is a reference that can be left unloaded; when loaded it composes exactly like a reference but ranks just below references. | composition/loading |
| 40 | Load and unload (working sets, load rules) | A stage can be opened with all, none or some payloads loaded, and prims can be loaded/unloaded later; unloading a prim unloads payloads beneath it. | composition/loading |
| 41 | Instanceable prims | A prim marked instanceable shares one prototype with every prim that composes the same way; overs inside an instance, or on its ancestors, are ignored, and its own arcs are kept. | instancing/native |
| 42 | Skinned instances | Instanced skinned characters share one prototype, yet each can play its own animation (skel:animationSource) and carry its own primvars. | instancing/crowds |
| 43 | Prototype sharing (which instances share a prototype) | Instances share a prototype only if they compose identically below the root (same arcs, same variant selections, same load state and mask); a different selection or an added class arc makes a new prototype. | instancing/native |
| 44 | Nested instancing and payloads inside instances | Prototypes can themselves contain instances, and instances can contain payloads; the same override restrictions apply at every level. | instancing/native |
| 45 | Path translation of targets across arcs | Relationship targets and attribute connections authored inside an asset are rewritten to the stage paths where things ended up after references, variants and relocates; targets that point outside what was brought in are dropped. | composition/namespace |
| 46 | Composition errors and recovery | Bad arcs (missing files, missing target prims, cycles, invalid relocates or targets, conflicting property types) are reported and skipped while the rest of the stage composes. | foundations/paths, composition/errors |

### 3. Values and time

| # | Feature | What it does | Where |
|---|---|---|---|
| 47 | Attribute value types (scalars, vectors, matrices, quaternions, strings, tokens, asset paths, arrays) | Every attribute has exactly one declared value type from a fixed list (bool, int/int64/uint/uint64/uchar, half/float/double, 2-4 vectors, 2-4 matrices, quaternions, string, token, asset, and arrays of all of them), and values must read back exactly as written in both text and binary files. | foundations/types, animation/keys |
| 48 | Value roles (point, normal, vector, color, texCoord, frame) | A role name such as color3f or point3f is the same numbers as float3 but tells tools how to interpret them (transform as a position, a normal, a direction, a linear color, a UV). | foundations/types |
| 49 | Time-valued attributes (timecode and duration) | timecode values are points in time and duration values are lengths of time; when a layer is offset or scaled, timecodes are offset and scaled while durations are only scaled. | foundations/files |
| 50 | Default values vs time samples (value resolution order) | An attribute can hold a timeless default and time samples at once; at a frame the strongest layer that has any value wins, and within that layer time samples (then splines) beat the default; asking with no time reads only the default. | animation/keys, stage/imageable, physics/motion |
| 51 | Interpolation of time samples (linear, held) and extrapolation | Between two keys USD blends linearly when the type allows it (component-wise for vectors and matrices, slerp for quaternions, element-wise for equal-length arrays) and otherwise holds the earlier key; before the first and after the last key it holds. | animation/keys |
| 52 | Attribute variability (uniform vs varying) | 'uniform' marks an attribute as not meant to change over time, but it is only a statement of intent: the core still reads and resolves its time samples. | foundations/model |
| 53 | Attribute blocks (None default) | Writing 'attr = None' in a stronger layer hides every weaker default and animation for that attribute, so it reads as having no authored value (the schema fallback, if any, shows through). | animation/blocks |
| 54 | Value blocks inside animation (blocked time samples and spline segments) | A None at one time sample (or a spline segment with no value) means 'no value from here until the next key', and unlike an attribute block the schema fallback does not show through. | animation/blocks |
| 55 | Animation blocks | Writing 'attr = AnimationBlock' hides weaker time samples and splines but lets a weaker default value through. | animation/blocks |
| 56 | Schema fallback values | A schema can give an attribute a value that is reported when nothing is authored (Sphere radius 1, visibility 'inherited'), without writing it to any file. | stage/schemas |
| 57 | Value clips (sequenced animation from many files) | 'clips' metadata on a prim pulls time samples for its attributes from a list of clip layers, each active over a stage-time range, as if the samples were authored in place. | foundations/model, animation/clips |
| 58 | Clip timing (times mapping, jumps, reversal, offsets) | The clip 'times' list maps stage time to time inside the clip, piecewise linearly, which allows holds, loops, reversal and jumps; layer offsets above the clips shift and scale the whole mapping. | animation/clips |
| 59 | Clip sets, template clips and clipSets ordering | A prim can have several named clip sets (each its own dictionary inside 'clips'), ordered by the list-edited 'clipSets' metadata; a set can also name its clips with a template path like 'sim.###.usd' and a start, end and stride. | animation/clips |
| 60 | Clip manifests and missing values in clips | A clip manifest lists which attributes the clips animate; when an active clip has no samples for one of them, USD uses the manifest's default or block, or, if asked (interpolateMissingClipValues), interpolates from neighbouring clips. | animation/clips |
| 61 | Splines (Ts animation curves) | An attribute can hold a spline of knots (time, value, tangents, per-segment interpolation: held, linear, curve; Bezier or Hermite), evaluated to a scalar value at any time. | foundations/files, animation/splines |
| 62 | Spline extrapolation and looping | Before the first and after the last knot a spline can be held, linear, sloped, or loop (repeat, reset, oscillate); 'inner loops' repeat a section in the middle (deprecated in 26.11). | animation/splines |
| 63 | Layer-offset timing of values (offset, scale, frame-rate scaling) | When a sublayer or reference carries an offset and scale, or a different timeCodesPerSecond, every time sample, spline knot and clip time it brings in is remapped to stage time. | animation/time |
| 64 | List ops (prepend, append, delete, explicit; legacy add/reorder) | List-valued fields (relationship targets, connections, references, inherits, apiSchemas, clipSets, and int/string/token list metadata) are edited per layer with prepend, append and delete, or reset with an explicit list, and the layers' edits combine into one ordered list with no duplicates. | foundations/files, composition/arcs, composition/namespace, composition/edits |
| 65 | Dictionary merging | Dictionaries (expressionVariables, clips, customData) merge key by key across layers: a stronger layer's key wins, a weaker layer's other keys survive. | foundations/model |
| 66 | Dictionary values and keys | A dictionary holds typed values (bool, int64, string, double2 pairs) under keys, quoted or not, and the empty string is a key too. | foundations/model |
| 67 | assetInfo | assetInfo is a dictionary describing where an asset came from (identifier, name, version); it records and changes nothing. | foundations/model |
| 68 | Relationships and target path translation | A relationship is a list-edited set of paths to other prims or properties; when it comes in through a reference, payload, variant or inherit, USD rewrites the authored paths into the stage's namespace. | foundations/paths, composition/namespace |
| 69 | Relationship forwarding | A relationship can target another relationship; asking for forwarded targets follows the chain to the final prims or properties, skipping purely forwarding relationships and stopping at cycles. | composition/namespace |
| 70 | Attribute connections | An attribute can carry a list-edited set of connection paths to other attributes (typically an input to an upstream output); USD stores and translates them but computes no dataflow. | foundations/paths, composition/namespace |
| 71 | Array edits (sparse edits to array values) | An array-valued attribute can hold an edit instead of a whole array (write value to index, append, prepend, erase, resize, minsize/maxsize) that applies on top of the next weaker array, including per time sample and in clips. | composition/edits |
| 72 | OpenExec (computed values) | Controllers compute attribute values, such as a joint's transform, from other attributes, so a rig drives its parts instead of every pose being keyed. | rigging/exec |

### 4. The scene: prims, schemas, transforms

| # | Feature | What it does | Where |
|---|---|---|---|
| 73 | Specifiers: def, over and class | Each prim spec says whether it defines a prim (def), only overrides one if it exists (over), or is an abstract template for inheriting (class); the composed prim takes the strongest def or class, and an over never changes it. | foundations/model, composition/arcs, stage/population |
| 74 | Active and inactive prims (non-destructive deletion) | Setting active = false on a prim removes it and everything under it from the composed stage, reversibly, and a stronger layer can set it back to true. | stage/population |
| 75 | Default traversal and prim flags | A normal walk of the stage visits only prims that are active, loaded, defined and not abstract; other walks can ask for inactive, unloaded, undefined or class prims explicitly. | foundations/model, stage/population |
| 76 | Typed and untyped prims (schema type names) | A prim's typeName (Mesh, Xform, Sphere...) names the schema that gives it built-in properties and behaviour; a prim may have no type, or a type this build doesn't know. | foundations/model, stage/schemas |
| 77 | Fallback prim types (fallbackPrimTypes) | Root-layer metadata that says 'if you don't know type X, treat the prim as type Y', tried in order. | stage/schemas |
| 78 | Applied API schemas (single-apply) | The apiSchemas list on a prim mixes extra schemas (MaterialBindingAPI, GeomModelAPI...) into its definition, adding built-in properties and answering HasAPI. | foundations/packages, stage/schemas |
| 79 | Multiple-apply API schemas (instance names) | Some API schemas (CollectionAPI, AccessibilityAPI, physics limits) can be applied several times to one prim, each with an instance name that becomes part of its property names. | stage/schemas |
| 80 | Prim definitions: built-in property metadata and metadata fallbacks | For built-in properties, the schema decides metadata such as variability and allowed tokens, and unauthored prim metadata (active, kind, instanceable) has a defined 'not set' meaning. | stage/schemas |
| 81 | Kind and model hierarchy (assembly, group, component, subcomponent) | The kind metadata labels prims as assemblies, groups, components or subcomponents, and the unbroken chain of group-kinds from the root down to components forms the model hierarchy, a cheap table of contents of the scene. | stage/population, stage/imageable |
| 82 | Model draw modes (origin, bounds, cards) | model:drawMode on a model asks viewers to replace the model's subtree with a stand-in (its axes, its bounding box, or textured cards); it inherits down the model hierarchy and applies at components unless model:applyDrawMode = false. | instancing/crowds, stage/population, stage/imageable |
| 83 | Purpose (default, render, proxy, guide) | Purpose tags imageable subtrees as always drawn (default), final-render only (render), fast-preview only (proxy) or interactive helpers (guide); an imageable prim's authored purpose wins, otherwise it inherits the nearest imageable ancestor's authored purpose. | stage/imageable |
| 84 | Visibility and its inheritance | visibility = invisible hides an imageable prim and its whole subtree (pruning); inherited (the fallback) means 'whatever my parent is'; it can be animated. | animation/keys, stage/imageable, stage/meaning |
| 85 | Transform ops | A prim's local transform is built from op attributes: translate, scale, rotateXYZ and the other rotation orders, orient (a quaternion), transform (a matrix). | foundations/types, geometry/xforms |
| 86 | Op order | xformOpOrder lists the ops in the order they apply; the same ops in another order place the prim differently. | geometry/xforms |
| 87 | Pivots (!invert! ops) | An !invert! op undoes a named op, so translate:pivot, a rotation, then !invert!translate:pivot turns a prim about a pivot point. | geometry/xforms |
| 88 | Extent on boundable prims | extent stores a gprim's local-space bounding box (min, max) for the values authored, so readers don't have to read all the points to bound it. | stage/imageable |
| 89 | Computed bounds and model extentsHint | World and local bounds of any subtree are computed from leaf extents and transforms for a chosen set of purposes, skipping invisible prims; models may cache their bounds per purpose in extentsHint. | stage/imageable |
| 90 | Primvars and interpolation modes | A primvar is an attribute in the primvars: namespace whose array is spread over the surface by its interpolation: constant (one value), uniform (per face), varying and vertex (per point), faceVarying (per face corner), with elementSize values per element. | stage/schemas, materials/primvars |
| 91 | Indexed primvars | A primvar can store unique values once plus an int[] primvars:name:indices array saying which value each element uses; flattening expands it. | materials/primvars |
| 92 | Primvar inheritance down namespace | Constant-interpolation primvars authored on an ancestor apply to every imageable descendant that doesn't author its own value. | materials/primvars |
| 93 | Collections (includes, excludes, expansion rules) | CollectionAPI:name on a prim defines a named set of prims and properties by including paths, excluding sub-paths, and an expansion rule (explicitOnly, expandPrims, expandPrimsAndProperties); collections can include other collections. | stage/collections |
| 94 | Point instancers and prototypes | A PointInstancer draws a copy of one of its prototypes at each point; protoIndices pick which, and the prototypes are not drawn where they stand. | instancing/points |
| 95 | Orientations and scales | Each instance can turn (orientations, quaternions) and stretch (scales) about its own origin. | instancing/points |
| 96 | Per-instance primvars | A primvar on the instancer with one value per instance, such as a color, gives every copy its own value. | instancing/points |
| 97 | Point instancer masking, ids and motion (inactiveIds, invisibleIds, velocities) | Instances can be hidden by id without editing the arrays (inactiveIds as list-edited metadata, invisibleIds as an animatable attribute), ids track particle identity over time, and velocities/accelerations give correct motion between samples. | instancing/points |
| 98 | Property ordering | A composed prim lists its properties in 'dictionary' order (case-insensitive, numbers compared as numbers), not as written, unless the strongest propertyOrder (reorder properties) moves some names to the front. | stage/population |

### 5. Domains: geometry, shading, lights, skeletons, physics and more

| # | Feature | What it does | Where |
|---|---|---|---|
| 99 | Holes | holeIndices name faces a mesh keeps in its topology but does not draw. | geometry/meshes |
| 100 | Normals | Authored normals shade a mesh smooth or sharp; a mesh with no normals and no subdivision draws faceted. | geometry/meshes |
| 101 | Double-sided | doubleSided draws a surface from behind as well as from the front; a single-sided surface's back may be culled. | geometry/meshes |
| 102 | Winding order | orientation (rightHanded or leftHanded) says which way each face's vertices wind, and so which side is its front. | geometry/meshes |
| 103 | Faces of any shape | faceVertexCounts and faceVertexIndices build faces with any number of corners, concave ones too, from one shared point list. | geometry/meshes |
| 104 | Subdivision schemes | subdivisionScheme says how a renderer smooths a mesh: catmullClark, loop (triangles), bilinear, or none (drawn as its polygons). | geometry/meshes |
| 105 | Creases and corners | Crease and corner sharpness keep chosen edges and vertices sharp while the rest of a subdivision surface smooths. | geometry/meshes |
| 106 | Implicit primitives (Sphere, Cube, Cone, Cylinder, Capsule, Plane and the _1 variants) | Analytic shapes defined by a few numbers (radius, height, size, axis) instead of a point list. | geometry/shapes |
| 107 | Curve widths | BasisCurves batch many curves into one prim; widths set how thick each is, per vertex or for the whole curve. | geometry/curves |
| 108 | Points | A Points prim stores particles, each drawn as wide as its width. | geometry/curves |
| 109 | Curve bases and wrap | A cubic curve follows its points by a basis (bezier, bspline, catmullRom); wrap makes it open, closed (periodic) or pinned to its end points. | geometry/curves |
| 110 | NURBS patches | A NurbsPatch is a smooth rational B-spline surface from a grid of control points, orders and knots. | geometry/curves |
| 111 | Hermite curves | HermiteCurves store points and tangents for interchange; a renderer need not draw them. | geometry/curves |
| 112 | Tetrahedral meshes | A TetMesh is a set of tetrahedra (4 indices each) plus optional surface triangles, for volumetric simulation. | geometry/meshes |
| 113 | Geometry subsets and families (incl. per-face materials) | GeomSubset child prims name sets of faces, points, edges, segments or tetrahedra of their parent; familyName and familyType group them, and the materialBind family gives per-face materials. | geometry/meshes, materials/binding |
| 114 | Motion: velocities, accelerations and motion blur | Point-based prims can carry velocities and accelerations so positions can be extrapolated between samples; MotionAPI's motion:blurScale and nonlinearSampleCount control blur. | animation/motion |
| 115 | Focal length and aperture | A perspective camera's focal length and film aperture set how much of the scene it sees at a distance (distance x aperture / focal). | cameras/lenses |
| 116 | Orthographic projection | An orthographic camera sees a fixed-size window with no perspective: parallel lines stay parallel, far things are not smaller. | cameras/lenses |
| 117 | Clipping range and clipping planes | A camera draws only between its near and far clipping distances, and only on the kept side of any clipping planes. | cameras/lenses |
| 118 | Focus distance and f-stop | focusDistance and fStop describe where a camera is sharp and how fast it blurs; a renderer with depth of field uses them. | cameras/lenses |
| 119 | Camera exposure | A camera's exposure, ISO, shutter and f-stop combine into one brightness scale for what it records. | cameras/render |
| 120 | Render contexts | A Material can carry a universal outputs:surface and renderer-specific ones such as outputs:mtlx:surface; each renderer picks its own. | materials/surfaces |
| 121 | Direct and inherited binding | material:binding on a prim binds its material to it and everything below it; a deeper binding wins. | foundations/paths, stage/schemas, materials/binding |
| 122 | Binding strength | An ancestor's binding marked strongerThanDescendants beats the bindings below it. | materials/binding |
| 123 | Material purpose | material:binding:preview and material:binding:full bind different materials for preview and final renders; the purpose's own binding beats the all-purpose one. | materials/binding |
| 124 | Collection-based material binding | material:binding:collection:<name> targets a collection and a material, so one relationship assigns a look to many prims. | stage/population, stage/schemas, stage/collections, materials/binding |
| 125 | UsdPreviewSurface | One small physically based surface every USD reader is expected to draw: diffuse, metallic, roughness, emission, opacity, ior and a few more inputs. | materials/surfaces |
| 126 | Texture coordinates (primvar readers, UsdTransform2d) | A primvar reader feeds a texture its st coordinates, and UsdTransform2d can rotate, scale and move them on the way. | materials/textures |
| 127 | Normal maps | A texture's scale and bias turn a tangent-space normal map's 0..1 values into normals the surface shades with. | materials/textures |
| 128 | Alpha cutouts | opacityThreshold cuts a surface where its opacity is below the threshold: a leaf, not a square. | materials/textures |
| 129 | Texture files | UsdUVTexture reads 8- and 16-bit PNGs and half-float EXRs as the same picture. | materials/textures |
| 130 | UDIM tiles | A texture path with <UDIM> reads one file per unit square of uv: 1001, 1002 and on. | materials/textures |
| 131 | MaterialX files | A .mtlx document can be referenced like a USD layer; its materials, node graphs and ND_ shaders appear as prims. | materials/surfaces |
| 132 | MaterialX nodes in USD | MaterialX shaders (ND_ nodes such as ND_standard_surface_surfaceshader) can be authored directly as USD Shader prims. | materials/surfaces |
| 133 | Color temperature | A light's color can be set as a blackbody temperature in kelvin (enableColorTemperature, colorTemperature) instead of an RGB color. | lights/emission |
| 134 | Exposure | A light's brightness is its intensity times 2 to the power of its exposure: +1 exposure doubles it. | lights/emission |
| 135 | Normalize | With normalize on, a light gives the same total light whatever its size: a bigger light is dimmer per square metre. | lights/emission |
| 136 | Shaping cones | ShapingAPI narrows a light to a cone (shaping:cone:angle, softness), so it lights a pool instead of everything around it. | lights/shaping |
| 137 | Dome lights (environment maps, texture formats, pole axis, DomeLight_1) | A DomeLight lights the scene from an environment texture at infinity; texture:format and (on DomeLight_1) poleAxis say how the image wraps the sphere. | lights/environment |
| 138 | Skeletons and joint animation | A Skeleton lists joints as path tokens with bind and rest transforms; a SkelAnimation supplies vectorized translations, quaternion rotations and scales over time; skel:skeleton and skel:animationSource bind them under a SkelRoot. | animation/characters |
| 139 | Skinning (joint influences, geomBindTransform, linear and dual-quaternion) | SkelBindingAPI primvars skel:jointIndices and skel:jointWeights (with elementSize) and skel:geomBindTransform tell how points follow joints, by classicLinear or dualQuaternion skinning. | instancing/crowds, animation/characters |
| 140 | Blend shapes and inbetweens | BlendShape prims hold point offsets (optionally sparse by pointIndices, with normalOffsets) and weighted inbetween shapes; the SkelAnimation's blendShapeWeights drive them by name. | animation/characters |
| 141 | Colliders | CollisionAPI marks the shapes a body collides with; here each collider is a guide-purpose copy of its body, so it shows only when guides are on. | physics/motion |
| 142 | Physics materials | Friction, restitution and density come from a physics material, bound with the physics purpose so it never changes how things look. | physics/motion |
| 143 | Mass and centre of mass | MassAPI gives a body its mass and centre of mass, which decide how it swings and falls. | physics/motion |
| 144 | Joints, drives and articulations | Joints connect two bodies and let them move only one way (slide, hinge, ball); a drive pushes along a joint, and an articulation root makes a chain one system. | physics/motion |
| 145 | Joint limits | A joint's limits stop it at a range: a revolute joint limited to -60..60 degrees never turns past its stops. | physics/motion |
| 146 | Render settings | RenderSettings say which camera renders, at what resolution and aspect, and how the picture fits the camera's aperture. | cameras/render |
| 147 | Render passes | A RenderPass renders a variant of the settings, for example with a collection of prims pruned out. | cameras/render |
| 148 | Spatial audio, asset previews and authorship (UsdMedia) | SpatialAudio places a sound in the scene (spatial or ambient) with playback timing; AssetPreviewsAPI stores thumbnails in assetInfo; AuthorshipAPI (new in 26.11) records who authored what. | animation/characters |
| 149 | Volume fields | A Volume names its fields (density, emission) through field:<name> relationships to OpenVDBAsset prims, each a file and a grid name. | geometry/volumes |
| 150 | Field sequences | A field's filePath can be time-sampled, so a volume plays a sequence of files. | geometry/volumes |
| 151 | Semantic labels (UsdSemantics) | Multiple-apply SemanticsLabelsAPI:<taxonomy> puts token lists of labels on prims; descendants accumulate their ancestors' labels. | stage/meaning |
| 152 | usdz for AR: anchoring, preliminary schemas and the web/AR-compliant profile | Apple's Preliminary_ schemas (AnchoringAPI, physics, behaviors, text) ride along as API schemas USD doesn't know, and usdchecker --arkit / usdzip --arkitAsset enforce a restricted profile: self-contained package, allowed file types, UsdPreviewSurface only, no nested gprims. | foundations/packages |
| 153 | Level of detail (UsdLod), new in 26.x | LODRootAPI marks a prim whose children are alternative representations; heuristic prims (distance, screen size) per domain (imaging, physics, audio) tell a renderer which to show; LODOverrideAPI forces a choice. | stage/meaning |

### Not in the museum yet

25 features: startTimeCode and endTimeCode; Layer documentation, comment and customLayerData; Default resolver, search paths and resolver contexts; Session layer; Edit targets; Layer muting; Saving and exporting layers; Flattening a stage; Flattening a layer stack; Layer identity, anonymous layers and reload; Relocates combined with classes, variants and payloads; Population masks; Namespace editing (move, rename, delete composed prims); Dynamic payloads (file format arguments from prim metadata); Asset info (assetInfo); resetXformStack (ignore parent transforms); Pattern-based collections (membership expressions and mode); Prim child ordering; Boundary and face-varying rules; NURBS curves; Materials, shaders, node graphs, connections and interface inputs; Shader sources (sourceAsset, sourceCode); Geometry and volume lights (MeshLightAPI, VolumeLightAPI, materialSyncMode); Light linking, shadow linking, light filters and filter linking; Particle fields.

<!-- features: end -->

## The museum

### Files

| File | What it is |
|---|---|
| [museum.usda](museum.usda) | The museum as one assembly: ten wings, each referenced from `./<wing>/<wing>.usda` and placed and timed along one route, plus the corridors between them |
| [tour.usda](tour.usda) | The guided tour, 16,542 frames (11.5 minutes): sublayers `museum.usda` and adds the camera, the load schedule and the doors closed between wings |
| [tour_cam.usdc](tour_cam.usdc) | The tour camera's path, referenced by `tour.usda` (binary: about 16,000 samples) |

The whole tour runs with OpenExec off, so it leaves out the Rigging wing, which is its own
tour (`rigging/tour.usda`).

### Wings

Eleven wings, in tour order:

| # | Wing | Galleries | Exhibits | Forgeries |
|---|---|---|---|---|
| 1 | [foundations](foundations/foundations.usda) | files, model, packages, paths, types | 8 | 53 |
| 2 | [composition](composition/composition.usda) | arcs, loading, strength, namespace, edits, errors | 9 | 97 |
| 3 | [instancing](instancing/instancing.usda) | native, points, crowds | 3 | 23 |
| 4 | [animation](animation/animation.usda) | keys, time, splines, clips, blocks, characters, motion | 7 | 72 |
| 5 | [rigging](rigging/rigging.usda) | exec | 2 | 9 |
| 6 | [stage](stage/stage.usda) | population, schemas, collections, imageable, meaning | 10 | 53 |
| 7 | [geometry](geometry/geometry.usda) | shapes, meshes, curves, volumes, xforms | 7 | 36 |
| 8 | [materials](materials/materials.usda) | surfaces, textures, binding, primvars, color | 6 | 37 |
| 9 | [lights](lights/lights.usda) | emission, shaping, environment | 4 | 11 |
| 10 | [cameras](cameras/cameras.usda) | lenses, render | 4 | 22 |
| 11 | [physics](physics/physics.usda) | motion | 2 | 7 |

62 exhibits and 420 forgeries. 410 forgeries are override layers; 10 are edited copies
because overrides can't express them (relocates, MaterialX and a few others; each says why).

Rigging is the one wing that runs with OpenExec on (its `tour.usda` says `exec = 1` in
`customLayerData`). It holds OpenExec's two exhibits, split out of Animation on Nov 9, so
Animation runs with exec off.

## Viewing

Open a file in OpenUSD's usdview and press Play. Tours play through their camera. Put
`config/variantFallbacks` on the plugin path first, for the one piece that needs it (see
Notes):

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
the tour camera, OpenExec and the workarounds in Notes set from the file, and the load
schedule in `tour.usda` applied as the camera moves.

## Notes

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
