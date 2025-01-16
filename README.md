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
  - pytorch3d
  - plotly
  - iopath
  - pytorch
  - nvidia
  - conda-forge
  - defaults
dependencies:
  - _libgcc_mutex=0.1=main
  - _openmp_mutex=5.1=1_gnu
  - anyio=4.1.0=pyhd8ed1ab_0
  - argon2-cffi=21.1.0=py39h3811e60_2
  - arrow=1.3.0=pyhd8ed1ab_0
  - asttokens=2.4.1=pyhd8ed1ab_0
  - async-lru=2.0.4=pyhd8ed1ab_0
  - attrs=23.1.0=pyh71513ae_1
  - babel=2.14.0=pyhd8ed1ab_0
  - beautifulsoup4=4.12.2=pyha770c72_0
  - blas=1.0=mkl
  - bleach=6.1.0=pyhd8ed1ab_0
  - brotli-python=1.0.9=py39h6a678d5_7
  - bzip2=1.0.8=h7b6447c_0
  - ca-certificates=2023.11.17=hbcca054_0
  - cached-property=1.5.2=hd8ed1ab_1
  - cached_property=1.5.2=pyha770c72_1
  - certifi=2023.11.17=pyhd8ed1ab_0
  - cffi=1.16.0=py39h5eee18b_0
  - charset-normalizer=2.0.4=pyhd3eb1b0_0
  - colorama=0.4.6=pyhd8ed1ab_0
  - cryptography=41.0.7=py39hdda0065_0
  - cuda-cudart=11.8.89=0
  - cuda-cupti=11.8.87=0
  - cuda-libraries=11.8.0=0
  - cuda-nvrtc=11.8.89=0
  - cuda-nvtx=11.8.86=0
  - cuda-runtime=11.8.0=0
  - decorator=5.1.1=pyhd8ed1ab_0
  - defusedxml=0.7.1=pyhd8ed1ab_0
  - entrypoints=0.4=pyhd8ed1ab_0
  - exceptiongroup=1.2.0=pyhd8ed1ab_0
  - executing=2.0.1=pyhd8ed1ab_0
  - ffmpeg=4.3=hf484d3e_0
  - filelock=3.13.1=py39h06a4308_0
  - fqdn=1.5.1=pyhd8ed1ab_0
  - freetype=2.12.1=h4a9f257_0
  - fvcore=0.1.5.post20221221=pyhd8ed1ab_0
  - giflib=5.2.1=h5eee18b_3
  - gmp=6.2.1=h295c915_3
  - gmpy2=2.1.2=py39heeb90bb_0
  - gnutls=3.6.15=he1e5248_0
  - idna=3.4=py39h06a4308_0
  - importlib-metadata=7.0.0=pyha770c72_0
  - importlib_metadata=7.0.0=hd8ed1ab_0
  - importlib_resources=6.1.1=pyhd8ed1ab_0
  - intel-openmp=2023.1.0=hdb19cb5_46306
  - iopath=0.1.9=py39
  - ipykernel=5.5.5=py39hef51801_0
  - ipython=8.18.1=pyh707e725_3
  - ipython_genutils=0.2.0=py_1
  - ipywidgets=8.1.1=pyhd8ed1ab_0
  - isoduration=20.11.0=pyhd8ed1ab_0
  - jedi=0.19.1=pyhd8ed1ab_0
  - jinja2=3.1.2=py39h06a4308_0
  - jpeg=9e=h5eee18b_1
  - json5=0.9.14=pyhd8ed1ab_0
  - jsonpointer=2.4=py39hf3d152e_3
  - jsonschema=4.20.0=pyhd8ed1ab_0
  - jsonschema-specifications=2023.11.2=pyhd8ed1ab_0
  - jsonschema-with-format-nongpl=4.20.0=pyhd8ed1ab_0
  - jupyter-lsp=2.2.1=pyhd8ed1ab_0
  - jupyter_client=8.6.0=pyhd8ed1ab_0
  - jupyter_core=5.5.0=py39hf3d152e_0
  - jupyter_events=0.9.0=pyhd8ed1ab_0
  - jupyter_server=2.12.1=pyhd8ed1ab_0
  - jupyter_server_terminals=0.5.0=pyhd8ed1ab_0
  - jupyterlab=4.0.9=pyhd8ed1ab_0
  - jupyterlab_pygments=0.3.0=pyhd8ed1ab_0
  - jupyterlab_server=2.25.2=pyhd8ed1ab_0
  - jupyterlab_widgets=3.0.9=pyhd8ed1ab_0
  - lame=3.100=h7b6447c_0
  - lcms2=2.12=h3be6417_0
  - ld_impl_linux-64=2.38=h1181459_1
  - lerc=3.0=h295c915_0
  - libcublas=11.11.3.6=0
  - libcufft=10.9.0.58=0
  - libcufile=1.8.1.2=0
  - libcurand=10.3.4.101=0
  - libcusolver=11.4.1.48=0
  - libcusparse=11.7.5.86=0
  - libdeflate=1.17=h5eee18b_1
  - libffi=3.4.4=h6a678d5_0
  - libgcc=7.2.0=h69d50b8_2
  - libgcc-ng=11.2.0=h1234567_1
  - libgomp=11.2.0=h1234567_1
  - libiconv=1.16=h7f8727e_2
  - libidn2=2.3.4=h5eee18b_0
  - libnpp=11.8.0.86=0
  - libnvjpeg=11.9.0.86=0
  - libpng=1.6.39=h5eee18b_0
  - libsodium=1.0.18=h36c2ea0_1
  - libstdcxx-ng=11.2.0=h1234567_1
  - libtasn1=4.19.0=h5eee18b_0
  - libtiff=4.5.1=h6a678d5_0
  - libunistring=0.9.10=h27cfd23_0
  - libwebp=1.3.2=h11a3e52_0
  - libwebp-base=1.3.2=h5eee18b_0
  - lz4-c=1.9.4=h6a678d5_0
  - markdown-it-py=3.0.0=pyhd8ed1ab_0
  - markupsafe=2.1.1=py39h7f8727e_0
  - matplotlib-inline=0.1.6=pyhd8ed1ab_0
  - mdurl=0.1.0=pyhd8ed1ab_0
  - mistune=3.0.2=pyhd8ed1ab_0
  - mkl=2023.1.0=h213fc3f_46344
  - mkl-service=2.4.0=py39h5eee18b_1
  - mkl_fft=1.3.8=py39h5eee18b_0
  - mkl_random=1.2.4=py39hdb19cb5_0
  - mpc=1.1.0=h10f8cd9_1
  - mpfr=4.0.2=hb69a4c5_1
  - mpmath=1.3.0=py39h06a4308_0
  - nbclient=0.8.0=pyhd8ed1ab_0
  - nbconvert-core=7.12.0=pyhd8ed1ab_0
  - ncurses=6.4=h6a678d5_0
  - nettle=3.7.3=hbbd107a_1
  - networkx=3.1=py39h06a4308_0
  - nodejs=6.13.1=0
  - notebook-shim=0.2.3=pyhd8ed1ab_0
  - numpy=1.26.2=py39h5f9d8c6_0
  - numpy-base=1.26.2=py39hb5e798b_0
  - openh264=2.1.1=h4ff587b_0
  - openjpeg=2.4.0=h3ad879b_0
  - openssl=3.0.12=h7f8727e_0
  - overrides=7.4.0=pyhd8ed1ab_0
  - packaging=23.2=pyhd8ed1ab_0
  - pandocfilters=1.5.0=pyhd8ed1ab_0
  - parso=0.8.3=pyhd8ed1ab_0
  - pickleshare=0.7.5=py_1003
  - pillow=10.0.1=py39ha6cbd5a_0
  - pip=23.3.1=py39h06a4308_0
  - pkgutil-resolve-name=1.3.10=pyhd8ed1ab_1
  - platformdirs=4.1.0=pyhd8ed1ab_0
  - plotly=5.18.0=py_0
  - plyfile=0.8.1=pyhd8ed1ab_0
  - portalocker=2.8.2=py39hf3d152e_1
  - prometheus_client=0.19.0=pyhd8ed1ab_0
  - ptyprocess=0.7.0=pyhd3deb0d_0
  - pure_eval=0.2.2=pyhd8ed1ab_0
  - pycparser=2.21=pyhd3eb1b0_0
  - pygments=2.17.2=pyhd8ed1ab_0
  - pyopenssl=23.2.0=py39h06a4308_0
  - pysocks=1.7.1=py39h06a4308_0
  - python=3.9.18=h955ad1f_0
  - python-dateutil=2.8.2=pyhd8ed1ab_0
  - python-fastjsonschema=2.19.0=pyhd8ed1ab_0
  - python-json-logger=2.0.7=pyhd8ed1ab_0
  - python_abi=3.9=2_cp39
  - pytorch=2.0.1=py3.9_cuda11.8_cudnn8.7.0_0
  - pytorch-cuda=11.8=h7e8668a_5
  - pytorch-mutex=1.0=cuda
  - pytorch3d=0.7.4=py39_cu118_pyt201
  - pytz=2023.3.post1=pyhd8ed1ab_0
  - pyyaml=6.0=py39hb9d737c_4
  - pyzmq=25.1.0=py39h6a678d5_0
  - readline=8.2=h5eee18b_0
  - referencing=0.32.0=pyhd8ed1ab_0
  - requests=2.31.0=py39h06a4308_0
  - rfc3339-validator=0.1.4=pyhd8ed1ab_0
  - rfc3986-validator=0.1.1=pyh9f0ad1d_0
  - rich=13.7.0=pyhd8ed1ab_0
  - send2trash=1.8.2=pyh41d4057_0
  - setuptools=68.2.2=py39h06a4308_0
  - six=1.16.0=pyh6c4a22f_0
  - sniffio=1.3.0=pyhd8ed1ab_0
  - soupsieve=2.5=pyhd8ed1ab_1
  - sqlite=3.41.2=h5eee18b_0
  - stack_data=0.6.2=pyhd8ed1ab_0
  - sympy=1.12=py39h06a4308_0
  - tabulate=0.9.0=pyhd8ed1ab_1
  - tbb=2021.8.0=hdb19cb5_0
  - tenacity=8.2.2=py39h06a4308_0
  - termcolor=2.3.0=pyhd8ed1ab_0
  - terminado=0.18.0=pyh0d859eb_0
  - tinycss2=1.2.1=pyhd8ed1ab_0
  - tk=8.6.12=h1ccaba5_0
  - tomli=2.0.1=pyhd8ed1ab_0
  - torchaudio=2.0.2=py39_cu118
  - torchtriton=2.0.0=py39
  - torchvision=0.15.2=py39_cu118
  - tornado=6.3.3=py39h5eee18b_0
  - tqdm=4.66.1=pyhd8ed1ab_0
  - traitlets=5.14.0=pyhd8ed1ab_0
  - types-python-dateutil=2.8.19.14=pyhd8ed1ab_0
  - typing_extensions=4.7.1=py39h06a4308_0
  - typing_utils=0.1.0=pyhd8ed1ab_0
  - uri-template=1.3.0=pyhd8ed1ab_0
  - urllib3=1.26.18=py39h06a4308_0
  - wcwidth=0.2.12=pyhd8ed1ab_0
  - webcolors=1.13=pyhd8ed1ab_0
  - webencodings=0.5.1=pyhd8ed1ab_2
  - websocket-client=1.7.0=pyhd8ed1ab_0
  - wheel=0.41.2=py39h06a4308_0
  - widgetsnbextension=4.0.9=pyhd8ed1ab_0
  - xz=5.4.5=h5eee18b_0
  - yacs=0.1.8=pyhd8ed1ab_0
  - yaml=0.2.5=h7f98852_2
  - zeromq=4.3.4=h2531618_0
  - zipp=3.17.0=pyhd8ed1ab_0
  - zlib=1.2.13=h5eee18b_0
  - zstd=1.5.5=hc292b87_0
  - pip:
      - addict==2.4.0
      - ansi2html==1.9.1
      - blinker==1.7.0
      - click==8.1.7
      - comm==0.2.0
      - configargparse==1.7
      - contourpy==1.2.0
      - cycler==0.12.1
      - dash==2.14.2
      - dash-core-components==2.0.0
      - dash-html-components==2.0.0
      - dash-table==5.0.0
      - flask==3.0.0
      - fonttools==4.46.0
      - itsdangerous==2.1.2
      - joblib==1.3.2
      - kiwisolver==1.4.5
      - matplotlib==3.8.2
      - nbformat==5.7.0
      - nest-asyncio==1.5.8
      - open3d==0.17.0
      - pandas==2.1.4
      - pexpect==4.9.0
      - prompt-toolkit==3.0.43
      - pymcubes==0.1.4
      - pyparsing==3.1.1
      - pyquaternion==0.9.9
      - retrying==1.3.4
      - rpds-py==0.15.2
      - scikit-learn==1.3.2
      - scipy==1.11.4
      - stack-data==0.6.3
      - threadpoolctl==3.2.0
      - tzdata==2023.3
      - werkzeug==3.0.1

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
   conda remove ffmpeg -y # --yes
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

<br>

### 6. Installing xvfb

xvfb is program that generate virtual monitor.
we use vessl server which is ubuntu environment without any GUI. But, COLMAP need GUI for execute.
<br>So, We can use this virtual monitor without a real monitor.

**Install xvfb:**

```bash
   sudo apt-get update
   sudo apt-get install xvfb
```


---


## Creating Input Data

**3dgs_sugar_unified** is root folder.


1. **Prepare folders:**
   - Place `my_video_input.mp4` inside `3dgs_sugar_unified/data/my_video/`.
   - Create `input/` folder. ⚠️ you must not name the other name, only "input" can be allowed.
   - Total path: `3dgs_sugar_unified/data/my_video/input`
   ```bash
   mkdir -p data/my_video/input_img
   ```

2. **Extract frames using FFmpeg:**
   ```bash
   ffmpeg -i ./data/my_video/my_video_input.mp4 -qscale:v 1 -qmin 1 -vf fps=10 ./data/my_video/input_img/%04d.jpg
   ```
   This saves approximately 160 image frames in `data/my_video/input_img/`.

3. **Run COLMAP to create sparse point cloud:**
   ```bash
   xvfb-run -a python convert.py -s data/my_video --no_gpu
   ```
    **Note:** "no_gpu" is temporarily issue. If you could execute without "--no_gpu", then you can remove "--no_gpu".

---

## Training Gaussian Splatting Model

1. Navigate to the Gaussian Splatting root folder:
   ```bash
   cd gaussian-splatting
   ```


2. Start training:
   ```bash
   python train.py -s <../data/my_video>
   ```

   A new folder will be created in `data/3dgs_output/`, containing model checkpoints and results.
   
   than, you have to return to root folder. `/3dgs_sugar_unified`
<br><br>

3. Rename the output folder for convenience: <br>
   The output folder-name is randomly generated. So, you can rename it with simple code.
   ```bash
   mv data/3dgs_output/<generated-folder-name> data/3dgs_output/<my_video>
   ```

---



[//]: # (## Changing the Output Folder Path)

[//]: # ()
[//]: # (By default, training results are saved to `/3dgs_sugar_unified/gaussian-splatting/output/`. If you want to move this folder to `/3dgs_sugar_unified/data/3dgs_output/`, run the following command:)

[//]: # (```bash)

[//]: # (  mv /3dgs_sugar_unified/gaussian-splatting/output/my_video /3dgs_sugar_unified/data/3dgs_output/my_video)

[//]: # (```)

[//]: # (Make sure to update the path accordingly after running `train_full_pipeline.py`. <br>)

[//]: # (because, we have to use this "output" for training SuGaR model to extract the mesh&#40;.obj&#41;.)

## Training and Model Configuration Guide

To train a SuGaR model, use the following comm**and:**

```bash
  cd SuGaR
  python train_full_pipeline.py -s <path to COLMAP dataset> -r <"dn_consistency", "density" or "sdf"> --high_poly True --export_obj True --gs_output_dir <path to the Gaussian Splatting output directory>
```
path to COLMAP dataset example => 3dgs_sugar_unified/data/my_video <br>
path to the Gaussian Splatting output directory example => 3dgs_sugar_unified/data/3dgs_output/my_video


### Regularization Method (-r Argument)
- Available options: "dn_consistency", "density", "sdf"
- Recommendation: `"dn_consistency"` (latest method, best mesh quality)
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
