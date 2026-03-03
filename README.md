# DNA–Lipid Interface Analysis: Multivalent Hydrophobicity

[![DOI](https://img.shields.io/badge/DOI-10.1021%2Facs.nanolett.4c02564-blue.svg)](https://doi.org/10.1021/acs.nanolett.4c02564)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

This repository contains the official computational pipeline for quantifying the membrane binding of DNA nanostructures to Giant Unilamellar Vesicles (GUVs) via multivalent hydrophobic anchoring, as published in *Nano Letters* (2024).

---

## Publication

> **Modulating the DNA/Lipid Interface through Multivalent Hydrophobicity**
> Siu Ho Wong et al.
> *Nano Letters* 2024, **24** (36), 11210–11218
> DOI: [10.1021/acs.nanolett.4c02564](https://doi.org/10.1021/acs.nanolett.4c02564)

## Journal Cover Recognition

The biophysical principles explored in this work were selected for the **Supplementary Cover** of *Nano Letters*, Volume 24, Issue 36 (September 11, 2024).

<p align="center">
  <img src="assets/cover_art.png" width="380" alt="Nano Letters Supplementary Cover, Vol. 24 Issue 36">
  <br><em>Nano Letters Supplementary Cover — Volume 24, Issue 36 (2024)</em>
</p>

<p align="center">
  <img src="assets/graphical_abstract.jpeg" width="680" alt="Graphical Abstract">
  <br><em>Graphical abstract: multivalent hydrophobic anchoring modulates DNA binding strength at the lipid bilayer interface.</em>
</p>

---

## Scientific Context

This work establishes a quantitative framework for the phase-independent attachment of DNA nanostructures to zwitterionic lipid bilayers through hydrophobic anchoring — an electrostatics-free mechanism where binding strength is precisely tuned by anchor multiplicity and structural orientation. The pipeline extracts per-vesicle Cy5 fluorescence intensities from confocal microscopy images, enabling rigorous statistical comparison across experimental conditions.

---

## Pipeline Overview

Confocal images contain two channels: the **NBD lipid channel** (Channel 0, used for segmentation) and the **Cy5 DNA channel** (Channel 1, used for quantification).

| Step | Stage | Method |
|------|-------|--------|
| 1 | Preprocessing | Gaussian smoothing of lipid channel (σ = `SIGMA_LIPID`) |
| 2 | Segmentation | Multi-population Otsu thresholding + binary closing |
| 3 | Labelling & Edge Exclusion | Connected-components labelling; border vesicles removed |
| 4 | Clump Exclusion (QC) | Touching/aggregated vesicles detected by dilation and discarded |
| 5 | Skeletonization | Morphological skeletonization + controlled dilation → thin membrane mask |
| 6 | Circularity & Size Filtering (QC) | Objects outside circularity range or below `MIN_PERIMETER` discarded |
| 7 | Quantification | Background-corrected mean and max Cy5 intensity per vesicle |

### Quality Control Filters

| Filter | Parameter | Default | Rationale |
|--------|-----------|---------|-----------|
| Minimum perimeter | `MIN_PERIMETER` | 240 px | Excludes sub-resolution debris |
| Circularity (lower) | `MIN_CIRCULARITY` | 0.2 | Rejects irregular, non-spherical objects |
| Circularity (upper) | `MAX_CIRCULARITY` | 1.0 | Retains well-formed vesicles |
| Edge exclusion | automatic | — | Removes partially imaged vesicles at frame borders |
| Clump exclusion | automatic | — | Removes vesicles in contact with neighbours |
| Out-of-focus | `THRESHOLD_OUT_OF_FOCUS` | 0.5 | Disabled by default; uncomment in config to activate |

### Output Columns (per vesicle)

| Column | Description |
|--------|-------------|
| `mean_dna_intensity` | Mean Cy5 intensity along the skeletonized membrane mask |
| `mean_dna_intensity_smoothed` | Mean Cy5 intensity using Gaussian-smoothed DNA channel (σ = `SIGMA_DNA`); noise-robust |
| `max_dna_intensity` | Peak Cy5 pixel value within the membrane mask |
| `mean_lipid_intensity` | Mean NBD lipid intensity (membrane reference) |
| `circularity` | 4π × area_filled / perimeter_contour² |
| `out_of_focus` | perimeter / area ratio; proxy for focus quality |
| `library` | Source subfolder (experimental condition) |
| `file_name` | Source image filename |
| `image_number` | Global image index across the batch |

---

## Setup & Usage

### 1. Requirements

- Python 3.10+
- `numpy`, `pandas`, `scikit-image`, `matplotlib`, `scipy`
- `pyclesperanto-prototype`
- `napari-simpleitk-image-processing`

### 2. Installation

```bash
git clone https://github.com/Herbert-Wong25/Multivalent_Biomolecular_Analysis.git
cd Multivalent_Biomolecular_Analysis
pip install -r requirements.txt
```

### 3. Data Setup & Execution

The pipeline uses relative paths and requires the following directory structure before execution:

```
data/
└── raw/
    ├── condition_A-d84/
    │   ├── image_001.ome.tif
    │   └── image_002.ome.tif
    └── condition_B-d84/
        └── image_001.ome.tif
```

- Each subfolder name must contain the `SUBFOLDER_FILTER` string (default: `"-d84"`). This can be changed in the configuration cell to match your own folder naming convention.
- Open and run **`Multivalent_Analysis_Pipeline.ipynb`** in Jupyter.
- Results are exported to `data/processed/Vesicle_Analysis_Results.csv`. A 4-panel diagnostic figure is saved per image.

### 4. Adjustable Parameters

All parameters are defined in a single **Configuration** cell at the top of the notebook — no need to search through the pipeline code.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `SUBFOLDER_FILTER` | `"-d84"` | Substring required in subfolder names to be processed |
| `SIGMA_LIPID` | `1` | Gaussian σ for lipid channel smoothing (segmentation) |
| `SIGMA_DNA` | `2` | Gaussian σ for DNA channel smoothing (intensity measurement) |
| `OTSU_CLASSES` | `3` | Number of intensity populations for multi-Otsu thresholding |
| `CLOSING_RADIUS` | `2` | Disk radius (px) for binary closing — fills membrane contour gaps |
| `DILATION_RADIUS` | `3` | Disk radius (px) for skeleton dilation — controls membrane mask thickness |
| `MIN_PERIMETER` | `240` | Minimum vesicle perimeter in pixels |
| `MIN_CIRCULARITY` | `0.2` | Lower circularity threshold |
| `MAX_CIRCULARITY` | `1.0` | Upper circularity threshold |
| `THRESHOLD_OUT_OF_FOCUS` | `0.5` | Out-of-focus filter threshold (disabled by default) |

---

## Repository Structure

```
Multivalent_Biomolecular_Analysis/
├── Multivalent_Analysis_Pipeline.ipynb   # Main analysis pipeline
├── requirements.txt
├── data/
│   ├── raw/                              # Input: confocal .ome.tif images
│   └── processed/                        # Output: CSV results + diagnostic figures
└── assets/
    ├── cover_art.png                     # Nano Letters Supplementary Cover
    └── graphical_abstract.png
```

---

## Contact

For questions regarding the methodology or to request access to the raw dataset, please contact **(Herbert) Siu-Ho Wong** — [herbert.wong150@gmail.com](mailto:herbert.wong150@gmail.com) · [LinkedIn](https://www.linkedin.com/in/siu-ho-wong-2020a)
