<div align="center">

# Gaussian Splatting: A Practical Guide for Data Preparation, Training, and Visualization

<font size="4">
Custom Implementation and Setup Guide
</font>
<br>

| [GitHub Repository](https://github.com/graphdeco-inria/gaussian-splatting) | [Documentation](https://graphdeco.inria.fr) | [Online Viewer](https://playcanvas.com/supersplat/editor) |

<img src="./media/playroom_hybrid.gif" alt="demo.gif" width="400"/>

</div>

## Overview

Gaussian Splatting is an advanced 3D reconstruction approach that requires image frames from multiple viewpoints and sparse 3D point cloud data to reconstruct detailed scenes. This document provides a step-by-step guide to setting up Gaussian Splatting, creating necessary input data, training on Ubuntu (Vessl server), and visualizing results using a Windows-based viewer.

---

## Prerequisites

### Ubuntu Environment:
- **OS**: Ubuntu 20.04
- **CUDA**: 11.4
- **GPU**: NVIDIA RTX 3090 (24GB RAM)



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

    <br> 1. clone gaussian-splatting

   ```bash
   git clone https://github.com/graphdeco-inria/gaussian-splatting --recursive
   ```  
   <br> 2. clone SuGaR

   ```bash
   git clone https://github.com/Anttwo/SuGaR.git --recursive
   ```
<br>
###
## Create and activate the Conda environment:
   ```bash
   python install.py
   conda activate gausugar
   ```

---




### 1. Installing and Reinstalling FFmpeg

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

---

### 2. Installing COLMAP and Verifying the Version

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

---

### 3. Installing and Verifying Imagemagick

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



## Data Preparation

Gaussian Splatting requires two types of input data:

1. **Images**: Collected from various angles.
2. **Sparse Point Cloud**: Spatial information associated with each image.

### Convert MP4 to Image Frames

FFmpeg can be used to convert video files into image frames:

```bash
ffmpeg -i input.mp4 -vf fps=1 frame_%04d.png
```

This command extracts one frame per second from the input video (`input.mp4`).

### Create Sparse Point Cloud Using COLMAP

COLMAP is used for Structure-from-Motion (SfM) and Multi-View Stereo (MVS):

1. Install COLMAP:
   ```bash
   sudo apt-get update
   sudo apt-get install colmap
   ```

2. Confirm installation:
   ```bash
   colmap -h
   ```

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

   Additionally, ensure that files like `cameras.bin`, `images.bin`, `points3D.bin`, and `project.ini` are correctly moved into `sparse/0/`.

---

## Troubleshooting Common Errors

### Error 1: Missing Shared Libraries
- **Error Message:** `libopenh264.so.5 cannot open shared object file`
- **Solution:**
  1. Remove all existing FFmpeg versions:
     ```bash
     sudo apt-get remove ffmpeg
     sudo apt-get purge ffmpeg
     conda remove ffmpeg
     ```
  2. Reinstall FFmpeg:
     ```bash
     sudo apt-get install ffmpeg
     ```
  3. Verify installation:
     ```bash
     ffmpeg -version
     ```

### Error 2: Missing 3D Reconstruction Data
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
   python train.py -s data/my_video
   ```

   A new folder will be created in `output/`, containing model checkpoints and results.

3. Rename the output folder for convenience:
   ```bash
   mv output/<generated-folder-name> output/my_video
   ```

---

## Visualizing Output on Windows

Since the viewer is an `.exe` file, visualization must be done on Windows:

1. **Transfer results to Windows:**
   ```bash
   scp -P <port_number> -r <user_id>@<ssh_ip>:/<path>/gaussian-splatting/output/my_video .
   ```

2. **Download the Viewer:**
   - Navigate to the [viewer download page](https://example-link).
   - Download the pre-built binaries and extract them.

3. **Run the viewer:**
   ```bash
   cd <viewer_folder>/bin
   SIBR_gaussianViewer_app.exe -m <path_to_output/my_video>
   ```

---

## Summary

This guide covers the complete workflow for data preparation, model training, and visualization using Gaussian Splatting. Ubuntu is used for data processing and training, while the results are visualized on a Windows machine using the provided viewer tool.

For additional support, refer to the [official repository](https://github.com/graphdeco-inria/gaussian-splatting) and the community discussion board.

---

## References
- [COLMAP Documentation](https://colmap.github.io/)
- [Gaussian Splatting GitHub](https://github.com/graphdeco-inria/gaussian-splatting)

<div align="center">

# Gaussian Splatting: A Practical Guide for Data Preparation, Training, and Visualization

<font size="4">
Custom Implementation and Setup Guide
</font>
<br>

| [GitHub Repository](https://github.com/graphdeco-inria/gaussian-splatting) | [Documentation](https://graphdeco.inria.fr) | [Pre-built Windows Viewer](https://example-link) |

<img src="./media/examples/demo.gif" alt="demo.gif" width="350"/>

</div>

## Overview

Gaussian Splatting is an advanced 3D reconstruction approach that requires image frames from multiple viewpoints and sparse 3D point cloud data to reconstruct detailed scenes. This document provides a step-by-step guide to setting up Gaussian Splatting, creating necessary input data, training on Ubuntu (Vessl server), and visualizing results using a Windows-based viewer.

---

## Prerequisites

### Ubuntu Environment:
- **OS**: Ubuntu 20.04
- **CUDA**: 11.4
- **GPU**: NVIDIA RTX 3090 (24GB RAM)

### Windows Environment:
- **OS**: Windows 10
- **CUDA**: 11.8
- **GPU**: NVIDIA GeForce RTX 3090 (24GB RAM)

### Required Libraries and Dependencies:

```yaml
name: gaussian_splatting
dependencies:
  - python=3.7.13
  - cudatoolkit=11.6
  - pytorch=1.12.1
  - torchaudio=0.12.1
  - torchvision=0.13.1
  - tqdm
  - plyfile
  - pip:
      - submodules/diff-gaussian-rasterization
      - submodules/simple-knn
      - submodules/fused-ssim
      - opencv-python
      - joblib
```

---

## Data Preparation

Gaussian Splatting requires two types of input data:

1. **Images**: Collected from various angles.
2. **Sparse Point Cloud**: Spatial information associated with each image.

### Convert MP4 to Image Frames

FFmpeg can be used to convert video files into image frames:

```bash
ffmpeg -i input.mp4 -vf fps=1 frame_%04d.png
```

This command extracts one frame per second from the input video (`input.mp4`).

### Create Sparse Point Cloud Using COLMAP

COLMAP is used for Structure-from-Motion (SfM) and Multi-View Stereo (MVS):

1. Install COLMAP:
   ```bash
   sudo apt-get update
   sudo apt-get install colmap
   ```

2. Confirm installation:
   ```bash
   colmap -h
   ```

---

## Setting Up the Environment

1. Clone the repository:
   ```bash
   git clone https://github.com/graphdeco-inria/gaussian-splatting --recursive
   ```

2. Create and activate the Conda environment:
   ```bash
   conda env create --file environment.yml
   conda activate gaussian_splatting
   ```

---

## Creating Input Data

1. **Prepare folders:**
   - Place `my_video_input.mp4` inside `gaussian-splatting/data/my_video/`.
   - Create `input/` folder:
   ```bash
   mkdir gaussian-splatting/data/my_video/input
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

   Additionally, ensure that files like `cameras.bin`, `images.bin`, `points3D.bin`, and `project.ini` are correctly moved into `sparse/0/`.

---

## Troubleshooting Common Errors

### Error 1: Missing Shared Libraries
- **Error Message:** `libopenh264.so.5 cannot open shared object file`
- **Solution:**
  1. Remove all existing FFmpeg versions:
     ```bash
     sudo apt-get remove ffmpeg
     sudo apt-get purge ffmpeg
     conda remove ffmpeg
     ```
  2. Reinstall FFmpeg:
     ```bash
     sudo apt-get install ffmpeg
     ```
  3. Verify installation:
     ```bash
     ffmpeg -version
     ```

### Error 2: Missing 3D Reconstruction Data
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
   python train.py -s data/my_video
   ```

   A new folder will be created in `output/`, containing model checkpoints and results.

3. Rename the output folder for convenience:
   ```bash
   mv output/<generated-folder-name> output/my_video
   ```

---

## Visualizing Output on Windows

Since the viewer is an `.exe` file, visualization must be done on Windows:

1. **Transfer results to Windows:**
   ```bash
   scp -P <port_number> -r <user_id>@<ssh_ip>:/<path>/gaussian-splatting/output/my_video .
   ```

2. **Download the Viewer:**
   - Navigate to the [viewer download page](https://example-link).
   - Download the pre-built binaries and extract them.

3. **Run the viewer:**
   ```bash
   cd <viewer_folder>/bin
   SIBR_gaussianViewer_app.exe -m <path_to_output/my_video>
   ```

---

## Summary

This guide covers the complete workflow for data preparation, model training, and visualization using Gaussian Splatting. Ubuntu is used for data processing and training, while the results are visualized on a Windows machine using the provided viewer tool.

For additional support, refer to the [official repository](https://github.com/graphdeco-inria/gaussian-splatting) and the community discussion board.

---

## References
- [COLMAP Documentation](https://colmap.github.io/)
- [Gaussian Splatting GitHub](https://github.com/graphdeco-inria/gaussian-splatting)


