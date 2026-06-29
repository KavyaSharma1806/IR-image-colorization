# ISRO BAH 2026 – Problem Statement 10

## Infrared Image Colorization & Enhancement for Improved Object Interpretation

---

# Vision

Develop a physically consistent, research-grade deep learning pipeline capable of converting single-channel Landsat 9 Thermal Infrared (Band 10 Level-1) imagery into high-quality, semantically accurate RGB imagery while simultaneously enhancing spatial resolution and preserving structural integrity.

This project aims not only to achieve strong benchmark metrics (PSNR, SSIM, FID), but also to build an interpretable, scalable remote sensing framework suitable for scientific and operational applications.

---

# Confirmed Facts

## Input

* Landsat 9 Level-1 Thermal Infrared Band 10
* Original sensor values (GeoTIFF)
* Single-channel image
* No visualization products
* No PNG/JPEG as model input

## Output

* Natural RGB image
* Ground Truth: Landsat Bands 4-3-2

## Training

* Self-prepared paired dataset
* Landsat 9 data from any official source
* Evaluation performed on hidden dataset during finale

---

# Core Philosophy

* Preserve physical meaning of sensor values.
* Never train on visualization images.
* Build a strong dataset before building a strong model.
* Prefer physically meaningful preprocessing over arbitrary image processing.
* Avoid hallucinating structures.
* Preserve semantic integrity.

---

# Project Roadmap

## Phase 1 — Remote Sensing Foundations

### Objectives

Understand how satellite imagery differs from conventional photography.

Topics

* Electromagnetic radiation in remote sensing
* Spectral signatures
* Landsat sensors (OLI & TIRS)
* Landsat band architecture
* Radiance vs Reflectance vs Brightness Temperature
* GeoTIFF and metadata
* Coordinate systems
* Atmospheric effects
* Image formation pipeline

Deliverable
Complete understanding of the physical sensing process.

---

## Phase 2 — Landsat Data Engineering

### Objectives

Create a high-quality supervised training dataset.

Tasks

* Download Landsat Level-1 scenes
* Scene filtering
* Cloud filtering
* QA band utilization
* RGB creation (Bands 4-3-2)
* Band 10 extraction
* Co-registration verification
* Patch extraction
* Train / Validation / Test split
* Metadata preservation

Deliverable
Research-quality paired dataset.

---

## Phase 3 — Preprocessing Research

Objectives

Investigate preprocessing methods rather than accepting defaults.

Topics

* Radiometric normalization
* Dynamic range analysis
* Histogram analysis
* Temperature scaling
* Outlier handling
* Patch overlap strategies
* Sensor degradation simulation
* Data augmentation
* Noise modelling

Goal

Determine preprocessing pipeline that maximizes generalization.

---

## Phase 4 — Super Resolution

Objective

Enhance thermal image resolution.

Models to Study

* SRCNN
* SRResNet
* EDSR
* RCAN
* SwinIR

Research Questions

* Should SR occur before colorization?
* Joint training vs independent training
* Realistic degradation modelling

Deliverable

Best-performing SR module.

---

## Phase 5 — Infrared Colorization

Objective

Translate Band 10 thermal imagery into natural RGB.

Candidate Models

* Pix2Pix
* CycleGAN
* Palette
* Diffusion Models
* ControlNet
* Conditional U-Net
* Vision Transformers

Research Questions

* Deterministic vs probabilistic colorization
* Semantic preservation
* Physics-aware constraints

Deliverable

High-fidelity RGB prediction model.

---

## Phase 6 — Semantic Guidance

Objective

Reduce unrealistic color assignments.

Possible Approaches

* Land-cover segmentation
* Semantic masks
* CLIP-based semantic consistency
* Multi-task learning
* Auxiliary segmentation loss

Goal

Ensure forests remain forests, water remains water, roads remain roads.

---

## Phase 7 — End-to-End Pipeline

Pipeline

Band 10
↓

Preprocessing
↓

Super Resolution
↓

Colorization
↓

Semantic Refinement
↓

Post Processing
↓

Final RGB

---

## Phase 8 — Evaluation

Metrics

Image Quality

* PSNR
* SSIM
* LPIPS
* FID

Task Performance

* Object Detection
* Semantic Segmentation

Performance

* Inference Time
* GPU Memory
* Model Size

Visual

* Human Inspection
* Failure Analysis

---

# Advanced Research Directions

Potential Improvements

* Physics-informed neural networks
* Multi-task learning
* Feature Pyramid Networks
* Transformer-based fusion
* Self-supervised pretraining
* Contrastive representation learning
* Knowledge distillation
* Test-time adaptation
* Uncertainty estimation
* Ensemble inference

---

# Repository Structure

project/

├── datasets/

├── preprocessing/

├── sr_models/

├── colorization_models/

├── semantic/

├── evaluation/

├── visualization/

├── notebooks/

├── configs/

├── experiments/

├── reports/

└── docs/

---

# Development Principles

* Reproducible experiments
* Version-controlled datasets
* Modular architecture
* Experiment logging
* Physics-aware preprocessing
* Explainable model decisions
* Extensive ablation studies
* Documentation for every experiment

---

# Success Criteria

Scientific

* Physically meaningful preprocessing
* Correct use of Landsat Level-1 data
* Strong remote sensing understanding

Engineering

* Clean modular codebase
* Automated preprocessing pipeline
* Efficient training and inference

Research

* Novel improvements beyond baseline
* Well-justified design choices
* Robust performance on unseen scenes

Competition

* High PSNR
* High SSIM
* Low FID
* Fast inference
* Strong qualitative results
* Excellent presentation and technical explanation

---

# Guiding Principle

Every preprocessing step, model choice, and architectural decision must answer one question:

**"Why is this the most scientifically and technically appropriate choice for Landsat thermal-to-RGB translation?"**
