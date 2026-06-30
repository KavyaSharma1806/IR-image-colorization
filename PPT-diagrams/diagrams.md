# ISRO BAH 2026 Idea Submission Template Guide
> **Screening Strategy:** This document details the precise text copy and the exact Mermaid visualizations required for each critical slide in the template to pass the automated technical screening round.

---

## SLIDE 3: Opportunity

### Slide Text Content
```text
The Baseline Limitation: Current state-of-the-art approaches treat thermal-to-visible translation as a pure graphic design/image-to-image style transfer problem. They process thermal bands as basic 8-bit pictures, ignoring sensor physics, which causes severe structural blur and hallucinated land objects.
How SMART-VIS Solves It: We break down the challenge into a modular pipeline where data is processed sequentially: Understanding -> Enhancement -> Reconstruction -> Verification.
Unique Selling Proposition (USP): Physics-Aware, Semantically-Guided Modular Inversion. SMART-VIS converts inputs directly into physical Brightness Temperature units (K) and introduces an invariant semantic tracking mask to anchor regional land boundaries. This mathematically prevents the network from hallucinating objects that do not exist.
```

### Slide Visual Component
```mermaid
graph LR
    subgraph Traditional Methods: Direct Image-to-Image Mapping
        T_Fail["Raw 8-bit Thermal Pic"] --> GAN_Fail["Standard Black-Box GAN"] --> Out_Fail["Visually Pleasing but Hallucinated Output<br/>(Fails Spatial Integrity)"]
    end

    subgraph SMART-VIS Strategy: Physics-Aware Modular Pipeline
        T_Succ["Raw Landsat 9 DN Matrix"] --> Phys["Physics Inversion (Kelvins)"] --> SR["Sub-Pixel Spatial Upscaler"] --> Sem["Semantic Context Anchor"] --> Out_Succ["Physically Plausible BGR GeoTIFF<br/>(Verified Spatial Accuracy)"]
    end

    style T_Fail fill:#fadbd8,stroke:#cd6155
    style Out_Fail fill:#fadbd8,stroke:#cd6155
    style T_Succ fill:#d4efdf,stroke:#27ae60
    style Out_Succ fill:#d4efdf,stroke:#27ae60
```

## SLIDE 4: List of Features Offered

### Slide Text Content
```text
Radiometric Preservation Module: Ingests raw .npy arrays and converts digital counts into real-world Top-of-Atmosphere (TOA) Spectral Radiance and Brightness Temperature (K), maintaining absolute sensor physics.
Multi-Scale Spatial Upscaler: Handles the explicit 200m -> 100m resolution scaling requirement using sub-pixel degradation backpropagation rather than generic bicubic resizing.
Parallel Semantic-Guidance Loop: Uses an internal deep feature extractor to cluster terrain signatures (Water, Veg, Urban, Soil) natively in the latent space to guide color assignments.
Cross-Modal Alignment Engine: Synthesizes the reflective characteristics of optical bands from emissive signatures.
GeoTIFF Compliance Formatter: Automatically structures outputs into the mandated output/model_outputs/ directories using strict Blue-Green-Red (BGR) channel sequencing.
```

## SLIDE 5: Process Flow Diagram / Use-Case Diagram

### Slide Visual Component
```mermaid
graph TD
    In["Input: Raw Landsat 9 Band 10 Array<br/>(200m Resolution Data)"] --> Prep["1. Radiometric Inversion Stage<br/>(DN --> TOA Radiance --> Brightness Temp)"]
    Prep --> SR["2. Spatial Super-Resolution Loop<br/>(Sub-Pixel Spatial Realism Engine)"]

    SR --> Out1["Output Deliverable 1: tir_superresolved_100m/<br/>(100m High-Fidelity GeoTIFF File)"]

    Out1 --> SemBranch["3. Parallel Semantic Context Masking<br/>(SegFormer Categorization Core)"]
    Out1 --> ReconBranch["4. Cross-Modal Waveform Mapping<br/>(Thermal Emittance --> Reflective Optical BGR)"]

    SemBranch --> Intersect["5. Semantic Refinement Boundary Validation<br/>(Eliminates Color-Bleeding and Boundary Blur)"]
    ReconBranch --> Intersect

    Intersect --> Post["6. Geospatial Affine Reconstruction<br/>(Restores Coordinate Systems & Map Projections)"]
    Post --> Out2["Output Deliverable 2: colorized_tir_100m/<br/>(100m Standard Multi-Band BGR GeoTIFF)"]

    style In fill:#ebf5fb,stroke:#2980b9,stroke-width:2px
    style Out1 fill:#fef9e7,stroke:#f39c12,stroke-width:2px
    style Out2 fill:#e8f8f5,stroke:#2ecc71,stroke-width:2px
```

## SLIDE 6: Implementation Methodology (Wireframes/Mocks)

### Slide Text Content
```text
Data Leakage Prevention Protocol: Scene-level spatial separation split (70% Train / 15% Val / 15% Test) to ensure zero patch-level spatial memorization during evaluation.
Patch Matrix Pipeline: Extracts overlapping 256 x 256 arrays from absolute physical units instead of compressed 8-bit images to maintain extreme radiometric bit-depth.
Verification Map Generation: Programmatic output of uncertainty verification maps alongside final BGR layers to flag unconfident pixel regions during downstream human analyst interpretation.
```

## SLIDE 7: Architecture Diagram of the Proposed Solution

### Slide Visual Component
```mermaid
graph LR
    subgraph Data Input Phase
        TIR_In["100m Super-Resolved TIR Tensor<br/>(From Stage 1 SR Module Output)"]
    end

    subgraph Solution Internal Pipeline
        direction TB
        subgraph Stream A: Feature Mapping
            Net_Gen["Attention U-Net Generator<br/>(Cross-Modal Wave Transfer Core)"]
        end
        subgraph Stream B: Structural Anchor
            Net_Sem["SegFormer Feature Encoder<br/>(Extracts Environmental Context)"]
        end
    end

    subgraph Multi-Objective Composite Loss Optimization Loop
        L1["L1 Distance Metric Loss<br/>(Coarse Alignment Guard)"]
        Adv["PatchGAN Discriminator Loss<br/>(High-Frequency Texture Detail)"]
        SemLoss["Cross-Entropy Invariance Loss<br/>(Enforces Strict Class-Color Boundaries)"]
    end

    TIR_In --> Net_Gen
    TIR_In --> Net_Sem

    Net_Gen -->|"Synthesized Visible Channels"| L1
    Net_Gen -->|"Synthesized Visible Channels"| Adv

    Net_Sem -->|"Class Anchored Masks"| SemLoss
    Net_Gen -->|"Synthesized Visible Channels"| SemLoss

    style TIR_In fill:#f4ecf7,stroke:#8e44ad,stroke-width:2px
    style SolutionInternalPipeline fill:#f2f3f4,stroke:#7f8c8d
    style SemLoss fill:#fadbd8,stroke:#cd6155,stroke-width:2px
```

## SLIDE 8: Technologies to be Used

### Slide Text Content
```text
Geospatial Processing Ecosystem: GDAL, Rasterio, Fiona (Ensures target GeoTIFF files preserve geospatial metadata matrices, map projection tags, and coordinate grids perfectly).

Deep Learning Framework: PyTorch, PyTorch Lightning (Powers training loops, handles parallel tensor computations, and standardizes multi-GPU execution pipelines).

Matrix & Image Engineering: NumPy (Enables processing directly on raw radiometric values inside .npy training patches), OpenCV, Albumentations (Provides robust, geospatial-safe data augmentation arrays).

Pre-trained Backbones & Tracking: HuggingFace Transformers (Accessing pre-trained remote sensing encoders), Weights & Biases (W&B) (Monitors ablation metrics and loss convergence stability).
```