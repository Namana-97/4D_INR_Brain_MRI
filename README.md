# 4D Implicit Neural Representation for Longitudinal Brain MRI

A continuous spatiotemporal neural field framework for modeling long-term anatomical changes in longitudinal brain MRI, going beyond discrete voxel-based approaches.

## Overview

Longitudinal brain imaging is essential for understanding gradual anatomical changes associated with aging, neurodegenerative disorders, and treatment monitoring. Traditional approaches treat each MRI scan as a discrete, isolated 3D observation, making it difficult to capture the smooth temporal evolution of brain morphology.

This project proposes a 4D Implicit Neural Representation (INR) framework that learns a continuous function:

```
f(x, y, z, t) -> MRI Intensity
```

This enables smooth reconstruction at arbitrary temporal locations, handles irregular scan intervals, provides a compact representation of anatomical change over time, and achieves better structural consistency than classical interpolation.

---

## Architecture

### Preprocessing Pipeline

- Convert to unified NIfTI format
- Apply z-score normalization
- Isotropic resampling to standardized resolution
- Rigid registration for longitudinal alignment
- Visual inspection across coronal, sagittal, and axial planes

---

### INR v1 - Subject-Specific Neural Field

```
Input: (x, y, z, t) coordinates
        |
Fourier Positional Encoding
        |
MLP [4 hidden layers + ReLU]
        |
Output: MRI Intensity Value
```

- Trained independently per subject
- Learns highly personalized anatomical trajectories
- Higher parameter count (one model per subject)

---

### INR v2 - Shared Multi-Subject Neural Field

```
Input: (x, y, z, t) + Subject ID
        |
Fourier Positional Encoding + Learned Subject Embedding
        |
Shared MLP [common structural patterns]
        |
Output: MRI Intensity Value
```

- Single shared model across all subjects
- Subject embeddings encode individual anatomical characteristics
- Fewer parameters with better generalization
- Outperforms INR v1 on PSNR and SSIM metrics

---

### Baseline Methods

| Method | Description |
|--------|-------------|
| Nearest Neighbor | Assigns the closest temporal neighbor's intensity |
| Linear Interpolation | Blends intensities between adjacent timepoints |
| Cubic Interpolation | Smooth curves using fixed mathematical kernels |

All baselines operate purely in the intensity domain and cannot model true anatomical evolution.

---

## Dataset

This project uses the OASIS-2 Longitudinal MRI Dataset, which contains multiple scans per subject collected over extended periods, covering nondemented and demented older adults.

Access: https://www.oasis-brains.org/

Place your result screenshots in the results/figures/ folder and reference them below:

```
![Sample MRI Slice](results/figures/sample_slice.png)
```

---

## Project Structure

```
4d-inr-brain-mri/
|
|-- data/
|   |-- raw/                    # Raw OASIS-2 NIfTI files
|   |-- processed/              # Preprocessed and registered scans
|   `-- splits/                 # Train/val/test subject splits
|
|-- models/
|   |-- inr_v1.py               # Subject-specific INR model
|   |-- inr_v2.py               # Shared multi-subject INR model
|   `-- positional_encoding.py  # Fourier positional encoding
|
|-- baselines/
|   `-- interpolation.py        # Nearest, linear, cubic baselines
|
|-- preprocessing/
|   `-- preprocess.py           # Normalization, registration, resampling
|
|-- evaluation/
|   `-- metrics.py              # PSNR, SSIM computation
|
|-- results/
|   |-- figures/                # PSNR/SSIM plots, residual maps
|   `-- final_accuracy_comparison.csv
|
|-- train.py                    # Training script
|-- evaluate.py                 # Evaluation script
|-- requirements.txt
`-- README.md
```

---

## Installation

### Prerequisites

- Python 3.8 or higher
- CUDA-compatible GPU (recommended)
- OASIS-2 dataset (requires free registration at https://www.oasis-brains.org/)

### Step 1: Clone the repository

```bash
git clone https://github.com/your-username/4d-inr-brain-mri.git
cd 4d-inr-brain-mri
```

### Step 2: Create a virtual environment

```bash
python -m venv venv
source venv/bin/activate
# On Windows: venv\Scripts\activate
```

### Step 3: Install dependencies

```bash
pip install -r requirements.txt
```

### Step 4: Prepare the dataset

Place your OASIS-2 NIfTI files in the `data/raw/` directory, then run:

```bash
python preprocessing/preprocess.py
```

---

## Usage

### Train INR v1 (Subject-Specific)

```bash
python train.py --model inr_v1 --subject OAS2_0001
```

### Train INR v2 (Shared Multi-Subject)

```bash
python train.py --model inr_v2 --subjects all
```

### Run Baselines

```bash
python baselines/interpolation.py --method cubic
```

### Evaluate All Methods

```bash
python evaluate.py --models all --metrics psnr ssim
```

---

## Results

### Quantitative Comparison

| Method | PSNR (dB) | SSIM |
|--------|-----------|------|
| Nearest Neighbor | 11.12 | 0.1086 |
| Linear Interpolation | 11.90 | 0.1005 |
| Cubic Interpolation | 11.90 | 0.1114 |
| INR v1 (Per-Subject) | 16.00 | 0.2140 |
| INR v2 (Shared) | 18.13 | 0.2109 |
What's up

### Key Findings

- INR-based methods consistently outperform all interpolation baselines across subjects.
- INR v2 achieves the highest PSNR (~18 dB) while using significantly fewer parameters.
- INR models preserve sharper cortical boundaries and reduce temporal blurring.
- Residual maps confirm INR reduces reconstruction error in anatomically dynamic regions.
- INR v2 requires approximately 8x fewer parameters than INR v1 across multiple subjects.

---

## Limitations

- Training on full volumetric data is computationally demanding, limiting accessible resolution.
- The framework is sensitive to registration quality; misalignment can introduce distortions that the neural field unintentionally encodes.
- The shared INR v2 struggles when subjects exhibit highly divergent anatomical patterns.
- Evaluation is limited to reconstruction and does not cover downstream predictive or diagnostic tasks.
- Future work may incorporate deformation priors, uncertainty modeling, and disease-specific temporal regularization.

---

## Team Contributions

| Member | Contributions |
|--------|--------------|
| Andrew Liu | Positional encoding integration and initial model pipeline setup |
| Nagalakshmi Cherukuri | Dataset curation, preprocessing, literature review, and registration quality verification |
| Namana | Interpolation baselines, PSNR/SSIM evaluation metrics, and quantitative comparisons |
| Pavan Kalyan Lingutla | INR v2 multi-subject embedding framework, parameter efficiency analysis, and model design |
| Sujan Uppalli Jayadevappa | Visualization, qualitative error analysis, reconstruction figures, and residual maps |

---

## References

1. Marcus et al. "Open Access Series of Imaging Studies (OASIS): Longitudinal MRI Data in Nondemented and Demented Older Adults." Journal of Cognitive Neuroscience, 2010.
2. Lorenzi and Pennec. "Efficient Longitudinal Registration for Modeling Brain Changes in Alzheimer's Disease." NeuroImage, 2019.
3. Feng et al. "Spatiotemporal Implicit Neural Representation for Unsupervised Dynamic MRI Reconstruction." arXiv, 2023.
4. Sitzmann et al. "Implicit Neural Representations with Periodic Activation Functions (SIREN)." NeurIPS, 2020.
5. Chen et al. "Neural Deformation Fields for Temporal Medical Image Registration." Medical Image Analysis, 2022.
6. Peng et al. "Longitudinal Prediction of Infant MR Images with Multi-Contrast GANs." Frontiers in Neuroscience, 2021.
7. Dannecker and Rueckert. "Predicting Longitudinal Brain Development via Implicit Neural Representations." MICCAI, 2024.
8. Aulakh et al. "Semi-Disentangled Spatiotemporal Implicit Neural Representations of Longitudinal Neuroimaging Data." arXiv, 2025.
9. Billot et al. "Robust Machine Learning Segmentation for Large-Scale Analysis of Heterogeneous Clinical Brain MRI Datasets." PNAS, 2023. Oh

---

Arizona State University -- CSE Research Project
