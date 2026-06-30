# ISRO BAH 2026 – Problem Statement 10

## Infrared Image Colorization & Enhancement for Improved Object Interpretation

---

# Vision

Develop a **physics-aware, semantically-guided, end-to-end deep learning pipeline** capable of converting single-channel Landsat 9 Thermal Infrared (Band 10 Level-1) imagery into high-quality, semantically accurate RGB imagery — while simultaneously enhancing spatial resolution and preserving structural integrity.

This project is built on a strict scientific philosophy: treat thermal sensor data as raw radiometric measurements, not as images. Every preprocessing step, model choice, and loss function must be physically and semantically justified.

---

# Confirmed Facts

## Input

* Landsat 9 Level-1 Thermal Infrared **Band 10** (TIRS, 10.6–11.19 µm)
* Raw GeoTIFF sensor values (Digital Numbers → converted to Brightness Temperature)
* Single-channel image, 100m spatial resolution
* **No PNG/JPEG as model input** — always raw GeoTIFF

## Output

* Natural RGB image equivalent
* Ground Truth: Landsat 9 **Bands 4-3-2** (Red-Green-Blue, 30m resolution)

## Training

* Self-prepared paired dataset (Band 10 ↔ Bands 4-3-2) from USGS Earth Explorer
* Evaluation performed on hidden dataset during finale
* Mixed-terrain scenes: urban, vegetation, water bodies, bare soil

---

# Core Philosophy

* **Preserve the physical meaning of sensor values** — convert DN to Brightness Temperature before any processing.
* **Never train on visualization images** — no PNGs or JPEGs as model input.
* **Build a strong dataset before building a strong model** — data quality determines model ceiling.
* **Prefer physically meaningful preprocessing** over arbitrary image processing.
* **Avoid hallucinating structures** — semantic guidance enforces correct color-object mapping.
* **Preserve semantic integrity** — water stays blue, forests stay green, always.
* **Every design choice must answer**: *"Why is this the most scientifically and technically appropriate choice for Landsat thermal-to-RGB translation?"*

---

# Chosen Architecture (Committed Decisions)

> These are our locked choices, selected for scientific rigor, 48-hour executability, and competitive differentiation.

| Component | Choice | Justification |
|---|---|---|
| **SR Model** | Real-ESRGAN (pretrained, fine-tuned on IR patches) | Best-in-class pretrained SR; adapts quickly to satellite data |
| **Colorization** | Pix2Pix with **Attention U-Net** Generator | Attention focuses on semantically important edges & boundaries |
| **Discriminator** | PatchGAN (70×70) | High-frequency texture realism without full-image overhead |
| **Semantic Guidance** | SegFormer (pretrained on remote sensing) | Generates land-cover conditioning masks fed into colorization |
| **Pipeline Order** | SR → Semantic Masking → Colorization → Post-processing | SR first gives cleaner features to the colorization model |
| **Loss Function** | GAN + L1 + VGG Perceptual + Semantic Constraint | Covers realism, structure, texture, and semantic correctness |

---

# End-to-End Pipeline

```
Landsat 9 Band 10 — Level-1 GeoTIFF (single-channel, 100m)
        |
        v
[Stage 0] PHYSICS-AWARE PREPROCESSING
        DN → Spectral Radiance → Brightness Temperature (Kelvin)
        QA Band cloud masking (< 10% cloud cover filter)
        Radiometric normalization + outlier clipping
        Patch extraction: 256x256 with 50% overlap
        Data augmentation: horizontal/vertical flip, rotation, noise injection
        |
        v
[Stage 1] SUPER-RESOLUTION — Real-ESRGAN (fine-tuned)
        Input:  1-channel thermal patch (100m effective)
        Output: 1-channel SR-enhanced thermal patch (30m equivalent sharpness)
        Recovers: faint edges, road boundaries, building outlines, water margins
        |
        +-------------------------------+
        |                               |
        v                               v
[Stage 2A] SR Thermal Patch     [Stage 2B] SEMANTIC LAND-COVER MASKING
        (1 channel)                     SegFormer (pretrained on remote sensing)
                                        Classes: Water | Vegetation | Urban | Bare Soil
                                        Output: 4-channel one-hot semantic mask
        |                               |
        +----------+  concatenate  +----+
                   v
        5-channel input (1ch SR thermal + 4ch semantic mask)
                   |
                   v
[Stage 3] SEMANTIC-GUIDED COLORIZATION — Pix2Pix cGAN
        Generator:     Attention U-Net (encoder-decoder with attention gates)
        Discriminator: PatchGAN (70x70 patch-level real/fake classification)
        Input:  5-channel concatenated tensor
        Output: 3-channel RGB patch
        |
        Loss Function:
        L_total = lambda_adv  * L_GAN           (adversarial — realism)
                + lambda_l1   * L_L1            (pixel-level — structure)
                + lambda_perc * L_VGG           (perceptual — texture / high-freq)
                + lambda_sem  * L_semantic       (semantic constraint — correct color-class mapping)
        |
        v
[Stage 4] POST-PROCESSING & TILE STITCHING
        Color histogram matching to Landsat Bands 4-3-2 reference distribution
        Overlap-weighted Gaussian tile blending (eliminates seam artifacts)
        Artifact detection and suppression
        Geospatial metadata restoration (CRS, spatial transform, projection)
        |
        v
FINAL OUTPUT — High-Fidelity RGB GeoTIFF
        Semantically accurate, artifact-free, full-scene RGB image
```

---

# Project Roadmap

## Phase 1 — Remote Sensing Foundations

### Objective
Understand how Landsat 9 thermal imagery differs from conventional photography before touching any code.

### Topics
* Electromagnetic radiation and thermal infrared sensing
* Spectral signatures of land-cover classes (water, vegetation, urban, soil)
* Landsat 9 sensor architecture — OLI (optical) vs TIRS (thermal)
* Band architecture: Band 10 (thermal) vs Bands 4-3-2 (natural RGB)
* Digital Numbers → Radiance → Brightness Temperature conversion chain
* GeoTIFF format, CRS, and spatial metadata
* Coordinate systems and reprojection
* Atmospheric effects on thermal imagery
* QA band interpretation for cloud masking

### Deliverable
Complete physics-grounded understanding of the sensing process and data format.

---

## Phase 2 — Landsat Data Engineering

### Objective
Build a research-quality, paired training dataset before any model work begins.

### Tasks
* Download Landsat 9 Level-1 scenes from **USGS Earth Explorer**
* Scene filtering: diverse terrain coverage (urban, vegetation, water, bare soil)
* Cloud filtering: QA band-based mask, retain scenes with < 10% cloud cover
* Band extraction: Band 10 (TIRS) and Bands 4-3-2 (OLI)
* Co-registration verification (Landsat 9 bands are natively co-registered)
* RGB composite creation from Bands 4-3-2
* Patch extraction: 256×256 tiles with 50% overlap
* Train / Validation / Test split (70 / 15 / 15) at scene level (no patch leakage)
* Metadata preservation per patch (scene ID, tile coords, cloud fraction)

### Deliverable
Paired dataset: `(Band10_patch.tif, RGB_patch.tif)` for each sample.

---

## Phase 3 — Physics-Aware Preprocessing

### Objective
Implement a radiometrically correct preprocessing pipeline — not arbitrary image normalization.

### Steps
* **Radiance conversion**: `L = M_L * Q_cal + A_L` (from MTL metadata)
* **Brightness Temperature**: `BT = K2 / ln(K1/L + 1)` (Kelvin)
* **Normalization**: Stretch BT to [0, 1] using scene-level percentile clipping (2nd–98th)
* **Dynamic range analysis**: histogram analysis per land-cover class
* **Outlier handling**: clip extreme values (instrument artifacts, emissivity anomalies)
* **Patch overlap strategies**: 50% overlap with Gaussian blending weights
* **Sensor degradation simulation**: add realistic noise for augmentation
* **Data augmentation**: random horizontal/vertical flip, 90° rotation, brightness jitter

### Goal
Preprocessing pipeline that maximizes generalization across diverse scenes and seasons.

---

## Phase 4 — Super-Resolution Module

### Objective
Enhance thermal image sharpness to recover faint spatial details before colorization.

### Chosen Model: Real-ESRGAN

**Why Real-ESRGAN over SRCNN/EDSR/SwinIR:**
* State-of-the-art pretrained weights available (xinntao/Real-ESRGAN)
* Realistic degradation modelling built-in (handles sensor noise, blur)
* Fine-tunable on domain-specific data (Landsat IR patches)
* Faster convergence from pretrained weights vs. training SRCNN from scratch

### Implementation
* Load pretrained Real-ESRGAN weights
* Fine-tune on paired: `(degraded_Band10_patch, sharp_Band10_patch)`
* Generate sharp thermal patches as input to Stage 3

### Research Questions (Resolved)
* **SR before or after colorization?** → **Before.** SR first gives the colorization model cleaner, higher-frequency input features.
* **Joint vs. independent training?** → **Independent** (modular, easier to debug in 48hrs; joint training as stretch goal).

### Deliverable
Fine-tuned Real-ESRGAN checkpoint producing 30m-equivalent sharpness from 100m Band 10.

---

## Phase 5 — Semantic Land-Cover Guidance

### Objective
Prevent unrealistic color assignments by conditioning colorization on per-pixel land-cover class.

### Chosen Model: SegFormer (pretrained on remote sensing)

**Why SegFormer:**
* Transformer-based architecture captures long-range spatial dependencies
* Pretrained weights available for remote sensing (GeoSeg / ADE20K-RS adapted)
* Lightweight inference (< 100ms per patch on GPU)
* 4-class output sufficient for our semantic constraint: Water | Vegetation | Urban | Bare Soil

### Implementation
* Run SegFormer inference on SR-enhanced thermal patches
* Output: 4-channel one-hot mask (one channel per class)
* Concatenate mask with SR thermal patch → 5-channel conditioning input to colorization model
* At training time: generate masks from Bands 4-3-2 ground truth (NDWI for water, NDVI for vegetation) as pseudo-labels

### Goal
Water always maps to blue. Forests always map to green. Roads/buildings always map to grey-brown.

---

## Phase 6 — Attention U-Net Colorization

### Objective
Translate SR-enhanced, semantically-conditioned thermal patches into natural RGB.

### Architecture: Pix2Pix with Attention U-Net Generator

**Generator — Attention U-Net:**
* Encoder: 7-layer downsampling path (Conv → BatchNorm → LeakyReLU)
* Attention Gates: applied at each skip connection between encoder and decoder
* Decoder: 7-layer upsampling path with skip connections (U-Net style)
* Input channels: 5 (1 SR thermal + 4 semantic mask)
* Output channels: 3 (RGB)

**Why Attention U-Net over vanilla U-Net:**
Attention gates suppress irrelevant background activations and focus learning on semantically important boundaries (water edges, road margins, building outlines) — directly addressing the low-contrast nature of thermal imagery.

**Discriminator — PatchGAN (70×70):**
* Classifies 70×70 overlapping patches as real or fake
* Penalizes high-frequency texture artifacts at local scale
* More computationally efficient than full-image discriminator

### Loss Function

```
L_total = lambda_adv  * L_GAN
        + lambda_l1   * L_L1
        + lambda_perc * L_VGG
        + lambda_sem  * L_semantic

Where:
  L_GAN      = adversarial loss (realism — fool discriminator)
  L_L1       = pixel-wise L1 (structural fidelity)
  L_VGG      = perceptual loss via VGG19 features (texture & high-freq details)
  L_semantic = cross-entropy between predicted class-colors and SegFormer mask classes

Suggested weights: lambda_adv=1, lambda_l1=100, lambda_perc=10, lambda_sem=10
```

### Training Configuration
* Optimizer: Adam (lr=2e-4, betas=(0.5, 0.999)) for both G and D
* Batch size: 8 (fits in 8GB VRAM at 256×256 patch size)
* Epochs: 200 (with early stopping on validation SSIM)
* Learning rate scheduler: Linear decay after epoch 100

### Deliverable
Trained Pix2Pix checkpoint producing 3-channel RGB from 5-channel conditioned input.

---

## Phase 7 — Post-Processing & Tile Stitching

### Objective
Reconstruct full-scene RGB output from patches without visible seams or color inconsistencies.

### Steps
* **Color histogram matching**: match predicted RGB histogram to Landsat Bands 4-3-2 reference using CDF equalization
* **Overlap-weighted Gaussian blending**: patches extracted with 50% overlap are blended with a Gaussian weight mask — eliminates hard seams
* **Artifact suppression**: detect and smooth localized color artifacts using median filtering
* **Geospatial metadata restoration**: write output GeoTIFF with original CRS, spatial transform, and band metadata

### Deliverable
Full-scene RGB GeoTIFF, geospatially correct, artifact-free.

---

## Phase 8 — Evaluation

### Image Quality Metrics

| Metric | Tool | What It Measures |
|---|---|---|
| **PSNR** | scikit-image | Peak Signal-to-Noise Ratio — pixel reconstruction quality |
| **SSIM** | scikit-image | Structural Similarity — structural integrity preservation |
| **LPIPS** | richzhang/PerceptualSimilarity | Perceptual similarity — human-aligned quality assessment |
| **FID** | pytorch-fid | Fréchet Inception Distance — realism of generated RGB |

### Performance Metrics

| Metric | Target |
|---|---|
| Inference time per tile | < 2 seconds on RTX 5050 (8GB) |
| GPU memory usage | < 6GB VRAM at batch size 8 |
| Model size | < 200MB total (SR + colorization) |

### Visual Evaluation
* Qualitative comparison: predicted RGB vs. ground truth Bands 4-3-2
* Failure case analysis: identify and document hallucination cases
* Per-class color accuracy: verify water=blue, vegetation=green, urban=grey

---

# Technology Stack

| Category | Tool / Library |
|---|---|
| **Language** | Python 3.10+ |
| **DL Framework** | PyTorch 2.x + torchvision |
| **Geospatial** | Rasterio, GDAL, pyproj |
| **Data Source** | USGS Earth Explorer / Google Earth Engine |
| **Super-Resolution** | Real-ESRGAN (xinntao/Real-ESRGAN) |
| **Segmentation** | HuggingFace Transformers — SegFormer |
| **Colorization** | Custom Pix2Pix (Attention U-Net + PatchGAN) |
| **Perceptual Loss** | VGG19 feature extractor (torchvision) |
| **Evaluation** | scikit-image (PSNR/SSIM), lpips, pytorch-fid |
| **Experiment Tracking** | Weights & Biases (W&B) |
| **Config Management** | Hydra / YAML |
| **Visualization / Demo** | Gradio or Streamlit |
| **Image Processing** | OpenCV, NumPy, Pillow |
| **Version Control** | Git + GitHub, DVC (dataset versioning) |

---

# Repository Structure

```
IR-image-colorization/
├── datasets/               # Raw scenes + preprocessed patch pairs
│   ├── raw/                # Downloaded Landsat 9 GeoTIFFs
│   ├── patches/            # Extracted 256x256 patch pairs
│   └── splits/             # train / val / test scene lists
├── preprocessing/          # Radiometric conversion, cloud masking, patching
├── sr_models/              # Real-ESRGAN fine-tuning code + checkpoints
├── colorization_models/    # Pix2Pix Attention U-Net + PatchGAN
├── semantic/               # SegFormer inference + mask generation
├── evaluation/             # PSNR, SSIM, LPIPS, FID scripts
├── visualization/          # Demo dashboard (Gradio/Streamlit)
├── notebooks/              # EDA, preprocessing validation, result analysis
├── configs/                # Hydra YAML configs (model, training, paths)
├── experiments/            # W&B run logs, training checkpoints
├── reports/                # Metric tables, comparison figures
└── Docs/                   # Idea submission, technical writeup
```

---

# Development Principles

* **Reproducible experiments** — every run logged with W&B, seeded RNG
* **Version-controlled datasets** — DVC tracks dataset versions alongside code
* **Modular architecture** — each stage (SR / semantic / colorization / eval) is independently runnable
* **Physics-aware preprocessing** — radiometric correctness is non-negotiable
* **No visualization artifacts** — all model inputs are raw sensor values, not rendered images
* **Ablation studies** — test each component's contribution individually (no SR, no semantic mask, no perceptual loss)
* **Failure documentation** — every hallucination case is logged and analyzed

---

# Advanced Research Directions (Stretch Goals)

* **Physics-informed loss** — incorporate brightness temperature constraints directly into the loss function
* **Transformer-based colorization** — replace Attention U-Net with a Vision Transformer (ViT) generator
* **CLIP-based semantic consistency** — use CLIP embeddings to measure semantic alignment between IR features and predicted colors
* **Multi-scale discriminator** — coarse-to-fine adversarial training for better global color coherence
* **Self-supervised pretraining** — pretrain encoder on large unlabeled Landsat corpus before colorization fine-tuning
* **Uncertainty estimation** — output confidence maps alongside colorized RGB (flag unreliable predictions)
* **Test-time adaptation** — adapt model to unseen geographic regions without retraining
* **Knowledge distillation** — compress full pipeline into a lightweight model for edge deployment

---

# Success Criteria

### Scientific
* Physically meaningful preprocessing (DN → BT, not arbitrary normalization)
* Correct use of Landsat Level-1 raw sensor data (no PNG inputs)
* Demonstrated understanding of thermal remote sensing physics

### Engineering
* Clean, modular, reproducible codebase
* Automated end-to-end pipeline (one command: raw GeoTIFF → colorized RGB)
* Efficient inference (< 2s per tile, < 6GB VRAM)

### Research
* Attention U-Net outperforms vanilla U-Net baseline (ablation evidence)
* Semantic conditioning reduces color hallucinations (ablation evidence)
* Composite loss outperforms L1-only baseline (ablation evidence)

### Competition
* High PSNR & SSIM (structural fidelity)
* Low FID (perceptual realism)
* Low LPIPS (human-aligned quality)
* Fast inference time
* Zero geospatial artifacts in stitched full-scene output
* Strong qualitative results across diverse terrain types

---

# Guiding Principle

Every preprocessing step, model choice, and architectural decision must answer one question:

**"Why is this the most scientifically and technically appropriate choice for Landsat 9 thermal-to-RGB translation?"**
