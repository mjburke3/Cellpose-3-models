# Cellpose 3 Nuclei Segmentation Models

Custom [Cellpose 3](https://www.cellpose.org/) machine-learning models for segmentation of nuclei in brightfield and fluorescent histological images of skeletal muscle and liver tissue.

> **Citation:** Burke MJ et al. *Manuscript in preparation.*

---

## Models

### 1. `10X_DAPI_Muscle_Nuc_V10`
**Fluorescent DAPI-stained skeletal muscle nuclei — 10X**

Trained to segment nuclei in fluorescence photomicrographs of DAPI-stained skeletal muscle. Trained in 10 human-in-the-loop cycles using consecutive 10X objective magnification images of tibialis anterior and quadriceps muscles from mdx mice.

| Property | Value |
|---|---|
| Modality | Fluorescence |
| Stain | DAPI |
| Tissue | Skeletal muscle (tibialis anterior, quadriceps) |
| Species | Mouse (mdx) |
| Magnification | 10X |
| Training cycles | 10 |
| Input | 3-channel RGB, 1920×1440 px |

---

### 2. `20X_HE_Muscle_Nuc_V27`
**H&E brightfield skeletal muscle nuclei — 20X**

Trained to segment hematoxylin-stained nuclei from greyscale brightfield images of H&E-stained skeletal muscle sections. Trained in 27 human-in-the-loop cycles using consecutive 20X objective magnification brightfield photomicrographs from dystrophic and wild-type canine skeletal muscle.

| Property | Value |
|---|---|
| Modality | Brightfield |
| Stain | Hematoxylin & Eosin (H&E) |
| Tissue | Skeletal muscle |
| Species | Canine (dystrophic + wild-type) |
| Magnification | 20X |
| Training cycles | 27 |
| Input | 3-channel RGB, 1920×1440 px |

---

### 3. `40X_Liver_Nuclei_UnmixColors_V20`
**Color-unmixed hepatic nuclei — 40X**

Trained to segment hepatic nuclei stained with Gill's #1 Hematoxylin. Images are preprocessed using the **CellProfiler UnMixColors module** to extract the hematoxylin channel from brightfield photomicrographs, producing fluorescence-like single-channel images on a dark background. The model was trained in 20 human-in-the-loop cycles on this color-extracted dataset.

| Property | Value |
|---|---|
| Modality | Color-unmixed brightfield |
| Stain | Gill's #1 Hematoxylin |
| Tissue | Liver (hepatic nuclei) |
| Preprocessing | CellProfiler UnMixColors (hematoxylin channel) |
| Magnification | 40X |
| Training cycles | 20 |
| Input | Single-channel greyscale, 1920×1440 px |

---

## Usage

### Requirements

```bash
pip install cellpose
```

### Loading a model

```python
from cellpose import models, io

# Load model (replace with path to downloaded model file)
model = models.CellposeModel(pretrained_model='path/to/10X_DAPI_Muscle_Nuc_V10')

# Load and run on an image
img = io.imread('your_image.tif')
masks, flows, styles = model.eval(img, channels=[0, 0])
```

### Preprocessing for `40X_Liver_Nuclei_UnmixColors_V20`

Images must be color-unmixed before running this model. In CellProfiler, use the **UnMixColors** module to extract the hematoxylin channel from brightfield photomicrographs prior to segmentation.

The following absorbance settings were used (3 channels to unmix):

| Channel Name | R | G | B |
|---|---|---|---|
| RedISH | 0.05 | 1 | 1 |
| Exclusion | 1 | 1 | 1 |
| Hematoxylin | preset: *"Hematoxylin and PAS"* | — | — |

The **Hematoxylin** channel output is the image passed to the model.

### Recommended parameters

These parameters were used during training and validation:

| Parameter | Value |
|---|---|
| `flow_threshold` | 0.4 |
| `cellprob_threshold` | 0.0 |
| `channels` | `[0, 0]` (greyscale) |

### Using with CellProfiler 4 — RunCellpose Plugin

These models can be used in CellProfiler 4 via the [RunCellpose plugin](https://plugins.cellprofiler.org/runcellpose.html). These models were run using the **Docker container method** (Docker image: `cellprofiler/runcellpose_no_pretrained:2.3.2`), which does not require a separate Cellpose installation.

To use a custom model in the RunCellpose module:

1. Find the **RunCellpose** module in the CellProfiler pipeline (or add it to a novel pipeline)
2. Under **Docker or Local Cellpose**, select **Docker**
3. Set the Docker image to `cellprofiler/runcellpose_no_pretrained:2.3.2`
4. Under **Model**, select **Custom model**
5. Browse to the locally downloaded model file and select the model (e.g. `10X_DAPI_Muscle_Nuc_V10`)
6. Set **Flow threshold** to `0.4` and **Cell probability threshold** to `0.0`
7. Set **Expected diameter** to `0` (auto-estimate) or use the mean diameters from the model training. These are found under **diameter** when loading the models to the Cellpose 3 GUI.  

> **Note for `40X_Liver_Nuclei_UnmixColors_V20`:** The **UnMixColors** module should be run upstream in the pipeline (see absorbance settings above) before passing the image to RunCellpose. 

---

## Training

Human-in-the-loop training was performed using the Cellpose 3 Python GUI, following the approach described by Stringer et al. Training used consecutive photomicrographs from each dataset with iterative correction of segmentation masks between cycles.

---

## References

- Stringer C, Wang T, Michaelos M, Pachitariu M. *Cellpose: a generalist algorithm for cellular segmentation.* Nature Methods, 2021. https://doi.org/10.1038/s41592-020-01018-x
- Pachitariu M, Stringer C. *Cellpose 2.0: how to train your own model.* Nature Methods, 2022. https://doi.org/10.1038/s41592-022-01663-4
- Stirling DR, Swain-Bowden MJ, Lucas AM, Carpenter AE, Cimini BA, Goodman A. *CellProfiler 4: improvements in speed, utility and usability.* BMC Bioinformatics 22, 433 (2021). https://doi.org/10.1186/s12859-021-04344-9
- Burke MJ et al. *Manuscript in preparation.*
