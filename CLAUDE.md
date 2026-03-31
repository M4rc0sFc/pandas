# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Single-notebook project (`Pandas.ipynb`) for **hyperspectral image analysis of vegetation plots**. The notebook processes hyperspectral raster data (.hdr/.raw ENVI format) from field parcels (HN2, JMM1, LFH2, LIMON1_3, LIMON2_4, TEJON2_6) and computes vegetation indices (NDVI, SAVI) with statistical summaries and ecological interpretation.

## Key libraries

- **numpy / pandas** — array manipulation and tabular stats
- **rasterio / spectral** — reading hyperspectral raster files (.hdr, ENVI)
- **matplotlib** — plotting spectra and index maps
- **python-magic** — file-type detection for input validation

## Notebook structure

1. **Data loading** — reads hyperspectral cubes from Google Drive paths, validates file types, stores as NumPy arrays
2. **Spectral regions** — parses .hdr headers, maps wavelength ranges (Blue, Green, Red, Red-Edge, NIR) to band indices
3. **NDVI computation** — calculates Normalized Difference Vegetation Index per parcel, with descriptive statistics and ecological interpretation
4. **SAVI computation** — calculates Soil-Adjusted Vegetation Index (L = 0.5) to correct for bare-soil effects in semi-arid areas
5. **Statistical analysis** — variance, std dev, coefficient of variation per parcel matrix; mean/median summaries

## Running the notebook

Originally developed in Google Colab. Data files live on Google Drive. To run locally, adjust the file paths in the "Ruta de archivos en Google Drive" section near the top of the notebook.

```bash
jupyter notebook Pandas.ipynb
```

## Conventions

- Comments and markdown cells are in **Spanish**
- Parcel names (HN2, JMM1, LFH2, LIMON1_3, LIMON2_4, TEJON2_6) are domain identifiers — do not rename
- Band slicing uses `(0, 400)` range — keep consistent when modifying spectral operations
- Known issue: TEJON_5 returns only 3 bands instead of 325 — documented in notebook, unresolved
