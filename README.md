<div align="center">

# 3DGS to Mesh: A Practical Guide for Data Preparation, Training, and Visualization

<font size="4">
Custom Implementation and Setup Guide
</font>
<br>

| [SuGaR Documentation](https://anttwo.github.io/sugar/) | [3D-GS Documentation](https://graphdeco.inria.fr) | [Online Viewer](https://playcanvas.com/supersplat/editor) |

<img src="./media/playroom_hybrid.gif" alt="demo.gif" width="500"/>

</div>

## Overview

Gaussian Splatting is an advanced 3D reconstruction approach that requires image frames from multiple viewpoints and sparse 3D point cloud data to reconstruct detailed scenes. This document provides a step-by-step guide to setting up Gaussian Splatting, creating necessary input data, training on Ubuntu (Vessl server), and visualizing results using a Windows-based viewer.

---

## Prerequisites

### Ubuntu Environment:
- **OS**: Ubuntu 20.04
- **CUDA**: 11.4
- **GPU**: NVIDIA RTX 3090 (24GB RAM)
- highly recommend to use **absolute route** for every step below



### Required Libraries and Dependencies:

```yaml
name: gausugar
channels:
  - pytorch
  - pytorch3d
  - nvidia
  - conda-forge
  - defaults
dependencies:
  - cudatoolkit=11.8  # support latest CUDA
  - python=3.9
  - pip
  - numpy
  - plyfile
  - tqdm
  - torchaudio=2.0.2
  - torchvision=0.15.2
  - pytorch=2.0.1
  - pytorch-cuda=11.8
  - pytorch3d=0.7.4
  - scipy
  - scikit-learn
  - jupyterlab
  - ipywidgets
  - open3d
  - plotly
  - matplotlib
  - nodejs
  - rich
  - fvcore
  - iopath
  - pyquaternion
  - xz
  - zlib
  - joblib
  - requests
  - ffmpeg  # for extract video frame
  - setuptools
  - ipython
  - dash
  - pandas
  - werkzeug
  - flask
  - libgcc-ng
  - libstdcxx-ng
  - pycparser
  - mkl
  - mkl-service
  - tqdm
  - pip:
      - addict
      - contourpy
      - pymcubes
      - nvdiffrast  # Mesh accelerate library
      - "submodules/diff-gaussian-rasterization"
      - "submodules/simple-knn"
      - "submodules/fused-ssim"
      - opencv-python

```

---

## Setting Up the Environment

### 1. Clone the repository:


   1. clone gaussian-splatting

   ```bash
      git clone https://github.com/graphdeco-inria/gaussian-splatting --recursive
   ```  
   <br> 2. clone SuGaR

   ```bash
      git clone https://github.com/Anttwo/SuGaR.git --recursive
   ```
<br>

### 2. Create and activate the Conda environment:

   ```bash
      python install.py
      conda activate gausugar
   ```


Gaussian Splatting requires two types of input data:

1. **Images**: Collected from various angles.
2. **Sparse Point Cloud**: Spatial information associated with each image.

<br>

### 3. Installing and Reinstalling FFmpeg

FFmpeg is used to convert videos into sequences of image frames. On Ubuntu, you can use `apt-get` to install FFmpeg.

**Steps for removing and reinstalling FFmpeg:**

1. Remove all existing FFmpeg installations:
   ```bash
   sudo apt-get remove ffmpeg
   sudo apt-get purge ffmpeg
   ```
2. Remove the FFmpeg module in the Conda environment:
   ```bash
   conda remove ffmpeg
   ```
3. Reinstall FFmpeg:
   ```bash
   sudo apt-get install ffmpeg
   ```
4. Check the version:
   ```bash
   ffmpeg -version
   ```

If you encounter the following error:

- "ffmpeg error while loading shared libraries: libopenh264.so.5 cannot open shared object file" Make sure to remove and reinstall all related libraries properly.

<br>

### 4. Installing COLMAP and Verifying the Version

COLMAP is a Structure-from-Motion and Multi-View Stereo tool used for feature matching across images, estimating camera positions, and generating 3D point clouds.

**Install COLMAP:**

```bash
   sudo apt-get update
   sudo apt-get install colmap
```

**Check COLMAP version:**

```bash
  colmap -h
```

Version example: `COLMAP 3.6 -- Structure-from-Motion and Multi-View Stereo`

<br>

### 5. Installing and Verifying Imagemagick

Imagemagick is a tool used for image processing, including format conversion and basic edits.

**Install Imagemagick:**

```bash
   sudo apt-get install imagemagick
```

**Check the version:**

```bash
   convert -version
```

Version example: `Version: ImageMagick 6.9.10-23 Q16 x86_64 20190101`

The above content has been added as the **Essential Preprocessing Tools and Installation Guide** in the README. This will help users convert videos into image sequences and process data efficiently using COLMAP and Imagemagick.


---


## Creating Input Data

1. **Prepare folders:**
   - Place `my_video_input.mp4` inside `3dgs_sugar_unified/data/my_video/`.
   - Create `input/` folder:
   ```bash
   mkdir 3dgs_sugar_unified/data/my_video/input
   ```

2. **Extract frames using FFmpeg:**
   ```bash
   ffmpeg -i gaussian-splatting/data/my_video/my_video_input.mp4 -qscale:v 1 -qmin 1 -vf fps=10 %04d.jpg
   ```
   This saves approximately 160 image frames in `data/my_video/input/`.

3. **Run COLMAP to create sparse point cloud:**
   ```bash
   xvfb-run -a python convert.py -s data/my_video
   ```

   **Note:** If COLMAP fails with an option error (`--Mapper.ba_global_function_tolerance`), modify `convert.py` by commenting out unnecessary flags:

   ```python
   mapper_cmd = (colmap_command + " mapper \
       --database_path " + args.source_path + "/distorted/database.db \
       --image_path " + args.source_path + "/input \
       --output_path " + args.source_path)
   ```

   Additionally, you can encount the error below  <br>
   Error => Missing 3D Reconstruction Data
   - **Error Message:** `No files found in data/my_video/distorted/sparse/0`
   - **Solution:**
     Move `data/my_video/0/` to `data/my_video/sparse/0/`:
   
        ```bash
        mv data/my_video/0/ data/my_video/sparse/0/
        ```

---

## Training Gaussian Splatting Model

1. Navigate to the Gaussian Splatting root folder:
   ```bash
   cd gaussian-splatting
   ```

2. Start training:
   ```bash
   python train.py -s <data/my_video>
   ```

   A new folder will be created in `output/`, containing model checkpoints and results.

3. Rename the output folder for convenience: <br>
   because, the output folder-name is randomly generated.
   ```bash
   mv output/<generated-folder-name> output/my_video
   ```

---



## Changing the Output Folder Path

By default, training results are saved to `/3dgs_sugar_unified/gaussian-splatting/output/`. If you want to move this folder to `/3dgs_sugar_unified/data/3dgs_output/`, run the following command:
```bash
  mv /3dgs_sugar_unified/gaussian-splatting/output/my_video /3dgs_sugar_unified/data/3dgs_output/my_video
```
Make sure to update the path accordingly after running `train_full_pipeline.py`. <br>
because, we have to use this "output" for training SuGaR model to extract the mesh(.obj).

## Training and Model Configuration Guide

To train a SuGaR model, use the following command:

```bash
  cd ..
  cd SuGaR
  python train_full_pipeline.py -s <path to COLMAP dataset> -r <"dn_consistency", "density" or "sdf"> --high_poly True --export_obj True --gs_output_dir <path to the Gaussian Splatting output directory>
```
path to COLMAP dataset example => 3dgs_sugar_unified/data/my_video <br>
path to the Gaussian Splatting output directory example => 3dgs_sugar_unified/data/3dgs_output/my_video


### Regularization Method (-r Argument)
- Available options: "dn_consistency", "density", "sdf"
- Recommendation: "dn_consistency" (latest method, best mesh quality)
- Suggested configurations based on experiments:
  - **Object-centered scenes:** "density"
  - **Scenes with complex backgrounds (e.g., Mip-NeRF 360 dataset):** "sdf"

### Polygon Configuration (--high_poly, --low_poly Arguments)
- `--high_poly True`: 1 million vertices, 1 Gaussian per triangle
- `--low_poly True`: 200k vertices, 6 Gaussians per triangle

### Refinement Time (--refinement_time Argument)
- Available options: "short" (2k iterations), "medium" (7k iterations), "long" (15k iterations)
- Default: "long" (15k iterations)
- "short" often provides good enough quality

### Export OBJ File (--export_obj Argument)
- Default setting: OBJ file export enabled
- Use case: Required for scene editing, merging, and animation in Blender

### Parameter Summary
| Parameter               | Type   | Description                                                                 | Default |
|-------------------------|--------|-----------------------------------------------------------------------------|---------|
| `--scene_path` / `-s`    | `str`  | Path to the source directory containing the COLMAP dataset.                 | N/A     |
| `--gs_output_dir`        | `str`  | Path to the checkpoint directory of a vanilla 3D Gaussian Splatting model. If not provided, training starts from scratch. | N/A     |
| `--regularization_type` / `-r` | `str`  | Type of regularization to use: "dn_consistency", "density", or "sdf". "dn_consistency" recommended. | N/A     |
| `--eval`                 | `bool` | Perform evaluation split of training images if set to True.                 | True    |
| `--low_poly`             | `bool` | Use low-poly mesh (200k vertices, 6 Gaussians per triangle) if set to True. | False   |
| `--high_poly`            | `bool` | Use high-poly mesh (1 million vertices, 1 Gaussian per triangle) if set to True. | False |
| `--refinement_time`      | `str`  | Refinement iteration time: "short" (2k), "medium" (7k), or "long" (15k). | "long" |
| `--export_ply`           | `bool` | Export a .ply file with 3D Gaussians at the end of training. File size can be large (~500MB), but necessary for 3DGS viewers. | True |
| `--export_obj` / `-t`    | `bool` | Export a traditional textured mesh as an .obj file from the refined SuGaR model. | True   |
| `--square_size`          | `int`  | Size of the square allocated to each pair of triangles in the UV texture. Reduce if memory issues occur. | 8 |
| `--white_background`     | `bool` | Set background to white if set to True.                                     | False   |


## Training Results and Viewer Usage

After training, results are saved in the `fined_mesh` and `refined_ply` directories.

- **Automatic .ply File Export:**
  After refinement, the `.ply` file containing the hybrid SuGaR representation is automatically saved in the `./output/refined_ply/` directory.
  This `.ply` file is compatible with any 3D Gaussian Splatting viewer.

The generated `.ply` and `.obj` files can be visualized using viewers or 3D modeling software such as Blender.

---




## Summary

This guide covers the complete workflow for data preparation, model training, and visualization using Gaussian Splatting. Ubuntu is used for data processing and training, while the results are visualized on a Windows machine using the provided viewer tool.

For additional support, refer to the **References** and the community discussion board.

---

## References
- [COLMAP Documentation](https://colmap.github.io/)
- [Gaussian Splatting GitHub](https://github.com/graphdeco-inria/gaussian-splatting)
- [SuGaR GitHub](https://github.com/Anttwo/SuGaR/)
