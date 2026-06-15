---
title: "A Comprehensive Guide to Installing TissUUmaps and Visualizing Visium HD Spatial Data"
date: 2026-06-15 12:00:00 +0000
categories: [Resources]
tags: [resources, tools]
author: Anjaney
---

Visium HD data offers incredible resolution, capturing spatial transcriptomics at the level of millions of 2µm bins or accurately segmented single cells. However, the sheer size of these datasets means you cannot simply load raw Space Ranger outputs into standard viewers. TissUUmaps is a powerful, Linux-native viewer that handles this scale using a pyramidal tiling engine (similar to Google Maps), which only loads the pixels and data points currently in view.

Installing complex scientific software like TissUUmaps from scratch—especially on newer operating systems like Ubuntu 24.04, can lead to deep dependency conflicts (often referred to as "DLL Hell"). This guide provides a highly reproducible, robust method for compiling and running TissUUmaps, along with the exact pre-processing steps required to visualize your Visium HD data.

### Part 1: The Installation Challenge and the Miniforge Solution

When installing TissUUmaps via standard `.deb` packages or `pip` on Ubuntu 24.04, you will likely encounter library mismatches. For example, PyInstaller-compiled binaries may fail with `undefined symbol: g_once_init_enter_pointer` due to conflicts with the newer GLib 2.80+. Furthermore, standard Python `h5py` installations might crash with an `Internal Server Error` (`ValueError: Not a datatype`) because the pre-compiled Python wheel expects HDF5 version 1.14.6, but Ubuntu 24.04 provides version 1.10.10. Additionally, TissUUmaps relies on an older version of Flask (2.x), while newer setups default to Flask 3.0, causing an `ImportError` for `_request_ctx_stack`.

To bypass these system-level conflicts entirely, the most reliable method is to use Miniforge (a lightweight Conda distribution). Conda creates a completely isolated environment that packages its own system libraries (`libhdf5`, `libvips`, `openslide`), avoiding Ubuntu's native libraries altogether.

#### Step 1: Install Miniforge

Run the following commands to download and install Miniforge:

```bash
wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh
bash Miniforge3-Linux-x86_64.sh -b
~/miniforge3/bin/conda init bash
source ~/.bashrc

```

#### Step 2: Create the Conda Environment and Install Dependencies

Create a clean Python 3.12 environment specifically for TissUUmaps:

```bash
conda create -n tissuumaps_env python=3.12 -y
conda activate tissuumaps_env

```

Next, install the "heavy" libraries via conda-forge. We explicitly install downgraded versions of Flask and Werkzeug to maintain compatibility with TissUUmaps:

```bash
conda install -c conda-forge h5py libvips openslide pyvips pytables flask=2.3.3 werkzeug=2.3.7 -y

```

Finally, install TissUUmaps via `pip`, but use the `--no-deps` flag to prevent it from overwriting the stable dependencies we just established via Conda:

```bash
pip install tissuumaps --no-deps

```

#### Step 3: Create a Desktop Shortcut

To avoid launching from the terminal every time, create a `.desktop` file. Note that you must replace `YOUR_USERNAME` with your actual Linux username.

```bash
nano ~/.local/share/applications/tissuumaps.desktop

```

Paste the following configuration:

```ini
[Desktop Entry]
Name=TissUUmaps
Comment=Spatial Transcriptomics Viewer
Exec=/home/YOUR_USERNAME/miniforge3/envs/tissuumaps_env/bin/tissuumaps
Icon=/home/YOUR_USERNAME/miniforge3/envs/tissuumaps_env/lib/python3.12/site-packages/tissuumaps/static/misc/logo.png
Terminal=false
Type=Application
Categories=Science;

```

Save (`Ctrl+O`, `Enter`) and exit (`Ctrl+X`). You can now launch TissUUmaps from your application menu.

### Part 2: Pre-processing Visium HD Data

Visium HD data must be pre-processed into a pyramidal image format and an AnnData (`.h5ad`) object before loading.

#### Preparing the High-Resolution Image

The `tissue_hires_image.png` generated in the Space Ranger `outs/` folder is merely a downsampled thumbnail (~2000 pixels wide) and will blur if you attempt to zoom to the single-cell level.

You must locate the full-resolution image. Look in `outs/spatial/` or `outs/image/` for a file named `cytassist_image.tiff` or `aligned_tissue_image.tif` (usually 300MB – 600MB). Alternatively, locate the raw slide scanner output (which can be 1GB to 20GB).

Convert this massive image into a Deep Zoom Image (DZI) using the command line tool `vips`. This breaks the image into tiles, allowing infinite zooming without memory crashes.

```bash
vips dzsave cytassist_image.tiff my_tissue_pyramid

```

#### Preparing the Expression Data (Segmented Cells)

Segmented cell outputs are generally biologically superior to arbitrary binned outputs (like 2µm or 8µm squares) because they aggregate data per actual cell, preventing signal mixing from neighboring cells.

You will need the `filtered_feature_cell_matrix.h5` and `cell_segmentations.geojson` files. Because loading hundreds of thousands of complex polygon shapes from a GeoJSON can cause severe lag, the standard workflow is to compute the central coordinates (centroids) of each cell and merge them into a fast-rendering AnnData `.h5ad` file.

Use the following Python script (run within an environment containing `scanpy` and `geopandas`) to perform this conversion:

```python
import scanpy as sc
import geopandas as gpd
import pandas as pd
import numpy as np

# 1. Load the gene expression data
adata = sc.read_10x_h5('outs/filtered_feature_cell_matrix.h5')

# 2. Load the complex cell shapes
gdf = gpd.read_file('outs/spatial/cell_segmentations.geojson')

# Fix Geometry Warning: Remove Coordinate Reference System
# The GeoJSON assumes map coordinates, but our slide is a flat Cartesian plane.
gdf.crs = None 

# 3. Calculate centroids
adata.obsm['spatial'] = np.column_stack([gdf.geometry.centroid.x, gdf.geometry.centroid.y])

# 4. Add Morphological Metadata (helps filter artifacts later)
adata.obs['area'] = gdf.geometry.area.values
adata.obs['circularity'] = (4 * np.pi * gdf.geometry.area) / (gdf.geometry.length ** 2)

# 5. Fix ID Formats (Integer to String)
# GeoJSON uses integer IDs (e.g., 119272), but H5 uses formatted strings.
gdf['formatted_id'] = gdf['cell_id'].apply(lambda x: f"cellid_{str(x).zfill(9)}-1")
adata.obs_names = gdf['formatted_id'].values

# 6. Save the optimized file for TissUUmaps
adata.write('visium_hd_segmented_centroids.h5ad')

```

*Brief Explanation:* The line `gdf.crs = None` is critical. Geopandas will throw a warning and calculate incorrect coordinates if it thinks your data is on a geographic sphere (Earth). Stripping the CRS forces it to use simple, flat Euclidean math suitable for a microscope slide. Furthermore, the `barcode_mappings.parquet` is not strictly necessary for this step since you can dynamically map the integer `cell_id` from the GeoJSON to the formatted string expected by the `.h5` file.

### Part 3: Running TissUUmaps and Visualizing the Data

Launch TissUUmaps using your desktop shortcut.

#### Loading the Data

1. 
**Load the Background Layer:** Drag and drop your converted `my_tissue_pyramid.dzi` (or `.tif`) directly into the viewer. This loads the foundation.


2. 
**Load the Markers Layer:** Navigate to `File > Load markers/regions` and select your newly created `visium_hd_segmented_centroids.h5ad` file. Note: If you are loading raw bin `.h5` files instead, you must apply the scale factor found in `scalefactors_json.json` (`tissue_hires_scalef`) so the micron coordinates match the pixel image.



#### Configuring the Visualization for Performance

Visium HD data sets are vast. Ensure your settings are optimized:

* Ensure the data is treated as "Markers", not "Regions".


* Select `spatial_0` for the X-coordinate and `spatial_1` for the Y-coordinate.


* Decrease the Marker size to between 5 and 10 pixels until you can distinctly see individual cell dots rather than a solid block of color.


* Lower the Opacity to ~0.6–0.8 to easily identify areas of dense cellular clustering.



#### Exploring Gene Expression and Filtering

To investigate specific gene expressions:

1. Navigate to the **Markers** tab.


2. In the `Color by` dropdown, manually type the gene name you wish to observe (e.g., `CD3E`, `INS`, `KRT`).


3. Switch the Colormap to something highly visible against dark backgrounds, such as `Viridis` or `Plasma`.



To refine your visualization:

1. Open the **Filter** (or Data) tab.


2. Select `area` to view a histogram. Drag the sliders to exclude extremely small points (debris) or extremely large points (segmentation errors or merged cells). The visualization will update in real-time.


3. You can similarly filter out cells with 0 expression for your target gene to achieve a perfectly clean map of localization.



*Alignment note:* Because we mapped raw coordinates onto the raw scanner image, minor rotations during the Space Ranger pipeline might cause the data points to drift from the tissue background. If the dots do not perfectly match the underlying H&E scan, navigate to the **Layers** tab and use the `Offset X` and `Offset Y` sliders to correct the alignment.
