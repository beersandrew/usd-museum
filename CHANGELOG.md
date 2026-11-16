# Changelog

Versions follow [semantic versioning](https://semver.org/): a major version when the tour's route or
the wings change shape, a minor version for new or reworked content and asset changes, a patch
version for fixes. Each version is a git tag (`v1.2.0`).

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
