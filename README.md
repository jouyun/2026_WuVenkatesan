# Wu & Venkatesan et al. — Protein Condensate Characterization by Fluorescence Microscopy

Code for the quantitative image analysis pipeline described in Wu, Venkatesan et al. (2026). This repository processes multi-channel fluorescence microscopy images to characterize protein phase separation behavior of TDP-43 variants and engineered fusion constructs (mEos3.1, BDFP, ELP tags).

## Overview

The pipeline segments individual cells from transmitted-light images using [Cellpose](https://github.com/MouseLand/cellpose), extracts per-cell fluorescence features across two channels, and quantifies colocalization and punctateness to classify condensate-forming behavior.

### Key measurements
- **Colocalization**: Pearson correlation coefficient between fluorescence channels (BDFP and mEos3.1), computed per cell with optional low-intensity pixel filtering
- **Punctateness**: Ratio of high-to-low intensity percentiles (e.g., p99/p50) within segmented cells, used as a proxy for condensate formation
- **Intensity statistics**: Per-cell mean intensity, percentiles (p50–p99), and standard deviation for each channel

## Notebooks

| Notebook | Description |
|----------|-------------|
| **A-BackgroundSubtraction_cluster.ipynb** | Core image processing pipeline. Performs morphological top-hat background subtraction, Cellpose segmentation on transmitted-light channel, label erosion, and per-cell feature extraction (intensity percentiles, Pearson correlation). Designed to run at scale on a SLURM cluster via Dask. |
| **B-Visualize.ipynb** | Data aggregation and visualization. Merges per-cell measurements with plate maps, filters by expression level, generates interactive box plots (Plotly) and regression plots (Seaborn) of Pearson correlation and punctateness across expression clones, performs K-means clustering, and runs Mann-Whitney U statistical tests. Includes Napari integration for interactive image inspection. |
| **C-AggregateFields.ipynb** | Aggregates per-field-of-view measurements across wells and plates into summary statistics. |

## Dependencies

- Python 3.x
- [Cellpose](https://github.com/MouseLand/cellpose) (GPU-accelerated cell segmentation)
- scikit-image
- OpenCV (`cv2`)
- NumPy, Pandas, SciPy
- Plotly, Seaborn, Matplotlib
- Dask + [dask-jobqueue](https://jobqueue.dask.org/) (for SLURM cluster execution)
- [Napari](https://napari.org/) (optional, for interactive visualization)
- tifffile

## Input Data

The pipeline expects multi-channel TIFF images organized in subdirectories by plate, with the following channel layout:
- **Channel 0**: BDFP fluorescence
- **Channel 1**: mEos3.1 fluorescence
- **Last channel**: Transmitted light (used for segmentation)

A `platemap.csv` file maps plate/row/column positions to expression clone identifiers and protein constructs.

## Usage

1. **Segmentation & feature extraction** — Run notebook A on a SLURM cluster (or adapt the Dask configuration for local execution). This produces per-image CSV files with cell-level measurements.
2. **Visualization & analysis** — Run notebook B to aggregate results, merge with plate maps, and generate figures.
3. **Field aggregation** — Run notebook C to produce well-level summary statistics.

