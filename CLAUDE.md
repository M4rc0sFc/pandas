# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project overview

**Author:** Marcos Guillermo Isunza Álvarez — Master's student, Facultad de Ciencias, UNAM.

**Thesis title:** *"Algoritmos de aprendizaje estadístico automatizado aplicados al análisis de imágenes hiperespectrales para el estudio de la biodiversidad vegetal"*

**Advisor:** Dr. Luis Daniel Ávila Cabadilla

The project analyzes hyperspectral imagery of **tropical dry forest** parcels (secondary and conserved) in Mexico to characterize functional plant diversity. The theoretical framework is the **Spectral Variation Hypothesis (SVH)**, which links spectral heterogeneity to biodiversity. Downstream objectives involve machine learning algorithms (KNN, Random Forest, hierarchical clustering).

---

## Repository structure

```
Pandas con Claude/
├── .claude/                       ← Claude Code configuration
├── .git/                          ← Git version control
├── Bitacora/                      ← Obsidian-format progress log (Markdown notes)
├── notasXiaomiPad7/               ← Field notes exported from tablet
├── putoLuisDaniel/                ← Working directory for Excel export notebook
│
├── Pandas.ipynb                   ← MAIN ANALYSIS NOTEBOOK (~24 MB)
│                                    Image loading, band extraction, vegetation indices,
│                                    heatmaps, descriptive statistics, ANOVA
│
├── putoluisdaniel.py              ← EXCEL EXPORT SCRIPT
│                                    Exports normalized hyperspectral bands to .xlsx
│                                    (one sheet per band with wavelength labels)
│
├── modificacionesExcel.py         ← Post-processing script for exported Excel files
│                                    (adds wavelength metadata to sheet names)
│
├── CLAUDE.md                      ← THIS FILE
├── Correcciones con Geógrafo.docx ← Dr. Roberto's normalization corrections log
├── altura,resolucion.xlsx         ← Flight altitude, GSD, and spatial resolution data
├── Area_Cubos_Recortados.xlsx     ← Post-crop area calculations per cube
├── normalización.png              ← Normalization pipeline diagram
└── borrador sv.jpg                ← Draft figure / sketch
```

### External resources (not in repo)

| Resource | Location |
|----------|----------|
| Hyperspectral cubes (ENVI raw + .hdr) | Google Drive: `/content/drive/MyDrive/Cubos2/` |
| Colab notebook (Pandas.ipynb) | [Link](https://colab.research.google.com/drive/18VjcZzRQwyt-w0BQysb8EqhAjnx4x-hj) |
| Colab notebook (putoLuisDaniel.ipynb) | [Link](https://colab.research.google.com/drive/1g_w8VCGJu0fm3yhUrQbGtoKpZUfyWOL9) |
| NDVI results Word doc | [Google Docs](https://docs.google.com/document/d/15qUpV-xh-6rEALed01-wvfbqmkwNBiyv/edit) |
| SAVI results Word doc | [Google Docs](https://docs.google.com/document/d/1nSRnLLBPxx-UMxImUWoQLIQE3A6PK_WU/edit) |
| Thesis document | `Tesis.docx` (in Claude.ai project files) |
| Progress tracker | `Bitácora.md` (in Claude.ai project files) |

---

## Sensor and data

- **Sensor:** Headwall Photonics Micro-Hyperspec VNIR, model uVS-206
- **Bands:** 325 spectral bands, range 380–1000 nm
- **Spectral resolution:** 1.8535 ± 0.0005 nm (uniform across all 7 valid cubes)
- **Pixel pitch:** 7.4 µm | **Focal length:** 8 mm | **IFOV:** 9.25 × 10⁻⁴ rad
- **Format:** ENVI flat binary (`.raw` + `.hdr` sidecar), opened with `spectral.open_image()`
- **Cropped cube dimensions:** (325 bands, 380 rows, 400 cols) = 152,000 pixels per cube
- **GSD range:** 8.76–11.65 cm/px (calculated as IFOV × H_AGL, with MSL→AGL correction)

---

## Study sites

| Code | Status | Notes |
|------|--------|-------|
| HN2 | ✅ Valid | Reference site; low heterogeneity |
| JMM1 | ✅ Valid | Forest parcel |
| LFH2 | ✅ Valid | Highest spatial heterogeneity (CV ~77%) |
| LIMON1_3 | ✅ Valid | Intermediate heterogeneity (CV ~43%) |
| LIMON2_4 | ⚠️ Valid but bugged | **BUG:** NDVI calculation uses NIR band from HN2 instead of LIMON2_4 |
| TEJON2_6 | ✅ Valid | Statistically similar to HN2 (Welch p=0.50) |
| UNAM | ✅ Valid | University parcel |
| TEJON_5 | ❌ **Excluded** | Only 3 bands — NOT a hyperspectral cube. Exclude from all spectral analyses |
| NAC2 | 🔶 Separate | Distinct cube; included in GSD/area calculations but not in main spectral analyses |

---

## Vegetation indices

| Index | Formula | Bands used | Status |
|-------|---------|------------|--------|
| NDVI | (NIR − Red) / (NIR + Red) | 172 (NIR), 131 (Red) | ✅ Complete (7 cubes) |
| SAVI | ((NIR − Red) / (NIR + Red + L)) × (1 + L), L=0.5 | Same | ✅ Complete (7 cubes) |
| NDRE | Uses Red-Edge bands | TBD | 🔄 In progress |
| EVI | Enhanced Vegetation Index | TBD | ⏳ Pending |
| NDWI | Normalized Difference Water Index | TBD | ⏳ Pending |
| SWIRI | Requires λ > 1000 nm | Beyond sensor range | ❓ Needs advisor decision |

---

## ⚠️ Critical knowledge

### Normalization destroys spectral signatures
Min-max normalization per band (used for Excel export and visualization) **destroys inter-band relationships** needed for vegetation indices like NDVI/SAVI. Always use raw digital numbers (DN) from `cubos_recortados` for index calculations, NOT from `normalized_*` arrays.

### GSD methodology
GSD = IFOV × H_AGL. The MSL-to-AGL correction is **critical** for high-elevation sites (LFH2, LIMON1_3, LIMON2_4, NAC2) — without it, GSD is overestimated by up to 3×. The thesis reports calculated GSD, not the ortomosaic resampled resolution.

### LIMON2_4 NDVI bug (unresolved)
```
nvdi_LIMON2_4 is computed with the correct normalized_LIMON2_4 bands,
BUT the print/verification block references nvdi_HN2 instead of nvdi_LIMON2_4.
This contaminates the ecological interpretation for LIMON2_4.
Fix: replace all nvdi_HN2 references in the LIMON2_4 code block with nvdi_LIMON2_4.
```

---

## Code conventions (Pandas.ipynb / pandas.py)

### Variable naming
- Raw cubes: `archivos` list (8 elements, indices 0–7)
- Cropped cubes: `cubos_recortados` (shape: 325 × 380 × 400)
- Normalized cubes: `normalized_HN2`, `normalized_JMM1`, etc.
- NDVI matrices: `nvdi_HN2`, `nvdi_JMM1`, etc. (note: `nvdi` without capital I — legacy typo, keep for consistency)
- SAVI matrices: `savi_HN2`, `savi_JMM1`, etc.
- CV vectors: `cv_HN2`, `cv_JMM1`, etc.

### Coding rules
1. Comments and markdown cells in **Spanish**
2. Prefer **dictionary-loop patterns** over repetitive per-cube code blocks
3. Use `ddof=1` for sample statistics (variance, std dev)
4. Always handle NaN values before computing statistics
5. Band slicing uses `(0, 400)` range — keep consistent
6. Parcel names (HN2, JMM1, etc.) are domain identifiers — do not rename

---

## Current state (April 2026)

### ✅ Completed
- NDVI and SAVI heatmaps + descriptive statistics for all 7 valid cubes
- Frequency distributions and ecological interpretation for NDVI/SAVI
- Word documents for NDVI and SAVI results
- Welch ANOVA on SAVI CV: F(5, 1097.3) = 376.1, ω² ≈ 0.26
- Metadata documentation (points 1–6): flight altitude, GSD, spectral mapping, spectral resolution, normalization corrections, post-crop areas
- Excel export of all normalized bands (putoLuisDaniel.ipynb)

### 🔄 In progress / Pending
- Fix `nvdi_LIMON2_4` NIR band bug
- Coefficient of variation as a function of wavelength
- NDRE implementation
- Complete metadata entry for TEJON_5 and NAC2
- Remaining vegetation indices (EVI, NDWI; SWIRI pending advisor decision)
- T statistic (Tᵢ = σ²ᵢ / σ²regional) as functional diversity indicator
- Machine learning phase: KNN, Random Forest, hierarchical clustering

### 📋 Thesis objectives timeline
- **Objective 1** (in progress): Spectral variation characterization — heatmaps, descriptive stats, comparative tables
- **Objective 2** (upcoming): ANOVA, T-statistic, rankings by spectral region
- **Objective 3** (future): ML algorithms — hierarchical clustering, KNN, Random Forest

---

## Key collaborators

| Person | Role |
|--------|------|
| Dr. Luis Daniel Ávila Cabadilla | Thesis advisor (also Marcos's uncle — referred to as "tío" or "Dr. Ávila") |
| Dr. Roberto | Normalization/radiometric corrections |
| Israel | Ortomosaic processing |

---

## Environment and tools

- **Execution:** Google Colab Pro (Python 3.12, 51 GB RAM, 8 CPU cores)
- **Storage:** Google Drive (`/content/drive/MyDrive/Cubos2/`)
- **Libraries:** `spectral`, `numpy`, `pandas`, `matplotlib`, `scipy`, `rasterio`, `python-magic`, `xlsxwriter`
- **Notes:** Obsidian vault at `C:\Users\Marco\Documents\Obsidian\Tesis Backup`
- **Local files:** `C:\Users\Marco\OneDrive\Documentos\TESISMARCOS`

---

## Communication preferences

- **Language:** Always respond in Spanish
- **Explanations:** Explain the *why*, not just the *how*
- **Code style:** DRY — prefer loops with dictionaries over repetitive blocks
- **Ecological context:** Always connect numerical results to their ecological meaning (functional diversity, spatial heterogeneity, vegetation condition)
- **Feedback:** Direct, unfiltered pushback when an idea is flawed — no agreement for politeness
- **Progress logging:** After completing important tasks, offer to update the Bitácora
