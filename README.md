# GLIM Manual Segmentation Editor

Standalone ROS 2 package for manually assigning semantic class labels to points in a GLIM dump map.

This package is intentionally kept outside the GLIM source tree. It links against GLIM, reuses the same viewer stack, and stores semantic annotations in a sidecar file so the original SLAM/map dump remains unchanged.

## Features

- Load a GLIM dump directory containing `graph.txt` and submap folders.
- Select points with the GLIM editor tools:
  - gizmo box/sphere selection
  - radius selection
  - outlier radius selection
  - MinCut segmentation
  - RegionGrowing segmentation
- Assign integer semantic class IDs to selected points.
- Edit class names and RGBA colors.
- Switch visualization between segmentation, intensity, Z height, and flat color.
- Save and reload labels from `<map_path>/semantic_labels.json`.
- Export a binary PLY with a per-point signed 32-bit `class` property.

## Build

From the ROS 2 workspace root:

```bash
colcon build --packages-select glim_manual_segmentation_editor --cmake-args -DPython3_EXECUTABLE=/usr/bin/python3
source install/setup.bash
```

The package defaults to `RelWithDebInfo`, matching GLIM's default optimized build type. To match a GLIM build that uses native CPU instructions:

```bash
colcon build --packages-select glim_manual_segmentation_editor --cmake-args -DPython3_EXECUTABLE=/usr/bin/python3 -DBUILD_WITH_MARCH_NATIVE=ON
```

## Run

```bash
ros2 run glim_manual_segmentation_editor manual_segmentation_editor /path/to/glim_dump
```

The map path argument is optional. If omitted, use `File > Open New Map` in the viewer.

## Basic Workflow

1. Load a GLIM dump directory.
2. Select points using the editor selection tools.
3. In `Semantic Labels`, choose a class ID, name, and color.
4. Click `Assign Selected`.
5. Use `File > Save Labels` to write `<map_path>/semantic_labels.json`.
6. Use `File > Export Semantic PLY` to export the full map with semantic labels.

Class `0` is reserved for `unlabeled`. Clearing selected labels resets those points to class `0`.

## Visualization Modes

The `Semantic Labels` panel has a `Visualization` selector:

- `Segmentation`: colors points by semantic class.
- `Intensity`: colors points by intensity with a TURBO colormap. Submaps without intensity use gray fallback.
- `Z`: colors points by world-frame height.
- `Flat color`: renders all map points with one editable color.

Labels can still be edited in any visualization mode. Switching back to `Segmentation` reapplies the semantic colors.

## Label Sidecar

Labels are stored in:

```text
<map_path>/semantic_labels.json
```

The file contains:

- format version
- class metadata: ID, name, RGBA color
- submap IDs and point counts
- run-length encoded class labels aligned to each submap's point order

Loading validates submap count and per-submap point counts before applying labels.

## PLY Export

`File > Export Semantic PLY` exports all loaded map points in world coordinates.

The exported PLY includes:

- all map vertices
- intensity, when every loaded submap has intensities
- `property int class`

Unlabeled points are exported as class `0`.

