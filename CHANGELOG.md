# Changelog

Versions follow [semantic versioning](https://semver.org/): a major version when the tour's route or
the wings change shape, a minor version for new or reworked content and asset changes, a patch
version for fixes. Each version is a git tag (`v1.2.0`).

## 1.3.0 — 2026-11-25

- Base labels: each pedestal, bench and base names what stands on it with one printed placard
  instead of two cut-out text quads. The placard is made the way the wall signs are: a paper card
  holding the title over the subtitle, sized to its text, on a dark backing board. Its texture is
  opaque with antialiased text and its material has no opacity or opacityThreshold, so the text
  edges are no longer stepped. 45 exhibits change; cut-outs that are part of an exhibit (swatches,
  stickers, the textures leaf) are unchanged.
- Every placard fits the face it is printed on: a line too long for a narrow pedestal is drawn
  smaller, and each card stays inside its face.
- The names exhibit's UTF-8 pedestal shows its name 日本 (it drew two missing-glyph boxes).
- The dome exhibit's pedestal has one placard per dome ("chrome ball" over the dome's name), shown
  while that dome lights the ball.
- Label atlases as RGBA8: 712 MB → 741 MB (the cards' paper margins). Files on disk: 54 MB → 53 MB.
- Thumbnails of the changed exhibits and wings, and the museum's, are re-rendered.

## 1.2.0 — 2026-11-16

- Label atlases: each atlas is the smallest sheet that holds its labels, at 900 px per metre (all
  atlases were 4096 px wide). 88 labels of exhibits that a forgery copies are now in atlases. Texture
  memory as RGBA8: 1,064 MB → 812 MB. Empty atlas area: 36% → 12%.
- The points and drawmodes signs (tour frames 3114 and 7369) are wider than 4096 px and were cropped
  on the right when packed into an atlas. Signs wider than an atlas now have their own texture.
- Removed 88 materials that no prim binds. Materials kept for lessons (unselected variants, binding
  alternatives, physics materials) and the hall doors' materials remain.
- The eight cart bodies' transforms and the tour camera's exposure responsivity, whose time samples
  were all equal, are authored as default values. Composed values are unchanged at every frame.
- The integer cart's points hold the cart at heights 1–5 instead of 0–8: 4,752 → 2,640 points.
  Composed geometry and the Storm warning the exhibit describes are unchanged.
- Joints exhibit: each ghost was the vane scaled 1.1× about the pivot, so the vane's head did not
  sit inside the ghost's. Each ghost is now the vane grown 1.1× about each part's centre and
  encloses the vane at its stop.
- Removed `st` from untextured balls, 252 zero-area faces from the suitcases, and 14 unreferenced
  images. The skinning exhibit's single-colour tubes author displayColor with constant
  interpolation.

## 1.1.0 — 2026-11-13

- `tour.usda` has cameras after `tour_cam`: one per wing, nine highlights, and one per exhibit stop.
  `/museum/tour` is the first child of `/museum`.
- The museum opens with every payload loaded, lit by a key and a fill distant light, with a
  table-of-contents namespace.
- extentsHint on every exhibit and hall entry.
- Label atlases: each exhibit's and hall's label images are in one texture.
- Removed the names exhibit's illegal-name cell (`def "1st"`). The README starts with what the
  museum is for and how to open it.

## 1.0.0 — 2026-11-12

First public version: wings of exhibits, each showing a USD feature next to forgeries (what a viewer
with the matching bug would draw), and a tour camera that passes every exhibit.
