# SMART-VIS — Mermaid Diagrams for Idea Submission
> Paste any diagram at: https://mermaid.live

---

## Diagram 1 — Full Pipeline Architecture
> Use for: Slide 5 (Process Flow) or Slide 7 (Architecture)

```mermaid
flowchart TD
    A(["Landsat-9 Band 10\nLevel-1 GeoTIFF\nSingle-channel Thermal"])

    subgraph PHY["PHYSICS LAYER — Preprocessing"]
        direction LR
        P1["DN to Radiance\nto Brightness Temp K"] --> P2["QA Band\nCloud Masking"] --> P3["Radiometric\nNormalization"] --> P4["256x256 Patch\nExtraction"]
    end

    subgraph SR["STAGE 1 — Super-Resolution"]
        S1["Real-ESRGAN\nFine-tuned on Landsat IR"]
        S1 --> S2(["SR-Enhanced Thermal Patch\n1-channel, 30m equivalent"])
    end

    subgraph SEG["STAGE 2 — Semantic Understanding (Parallel)"]
        G1["SegFormer\nPretrained on Remote Sensing"]
        G1 --> G2(["4-Class Land-Cover Mask\nWater, Vegetation, Urban, Soil"])
    end

    subgraph COL["STAGE 3 — Cross-Modal Reconstruction"]
        direction TB
        C1["Attention U-Net Generator\n5-ch input, 3-ch RGB output"]
        C2["PatchGAN Discriminator\n70x70 local texture"]
        C3["Loss = L_GAN + L_L1 + L_VGG + L_Semantic"]
        C1 <-->|adversarial| C2
        C1 --- C3
    end

    subgraph POST["STAGE 4 — Refinement"]
        direction LR
        R1["Histogram\nMatching"] --> R2["Gaussian Tile\nBlending"] --> R3["Geo-metadata\nRestoration"]
    end

    subgraph EVAL["EVALUATION"]
        direction LR
        E1["PSNR"] & E2["SSIM"] & E3["LPIPS"] & E4["FID"] & E5["Inference Time"]
    end

    A --> PHY
    PHY --> SR
    PHY --> SEG
    S2 --> CAT{"Concatenate\n5-ch Input"}
    G2 --> CAT
    CAT --> COL
    COL --> POST
    POST --> OUT(["Trustworthy RGB GeoTIFF\nGeospatially Correct, Full Scene"])
    OUT --> EVAL

    style PHY fill:#1a3a5c,color:#fff,stroke:#4a90d9
    style SR fill:#1a4731,color:#fff,stroke:#2ecc71
    style SEG fill:#3d1a5c,color:#fff,stroke:#9b59b6
    style COL fill:#5c1a1a,color:#fff,stroke:#e74c3c
    style POST fill:#5c4200,color:#fff,stroke:#f39c12
    style EVAL fill:#1a3a3a,color:#fff,stroke:#1abc9c
```

---

## Diagram 2 — SMART-VIS Reasoning Layers
> Use for: Slide 5 (Process Flow)

```mermaid
flowchart TB
    A["Observation Layer\nLandsat-9 Band 10\nRaw Thermal GeoTIFF"]
    B["Physics Layer\nDN to Brightness Temperature\nCloud Masking and Normalization"]
    C["Understanding Layer\nScene Representation Learning"]
    D1["Spatial Understanding\nReal-ESRGAN SR"]
    D2["Semantic Understanding\nSegFormer Masks"]
    D3["Physical Consistency\nRadiometric Check"]
    E["Cross-Modal Feature Fusion\n5-channel Conditioning"]
    F["Reconstruction Layer\nAttention U-Net + PatchGAN\nAdversarial + Perceptual Loss"]
    G["Refinement Layer\nHistogram Matching\nTile Stitching, Artifact Removal"]
    H["Intelligence Layer\nTrustworthy RGB Output\nPSNR, SSIM, LPIPS, FID"]

    A --> B --> C
    C --> D1 & D2 & D3
    D1 & D2 & D3 --> E
    E --> F --> G --> H

    style A fill:#D6EAF8,stroke:#1F618D,stroke-width:2px,color:#000
    style B fill:#FCF3CF,stroke:#B7950B,stroke-width:2px,color:#000
    style C fill:#EBDEF0,stroke:#7D3C98,stroke-width:2px,color:#000
    style D1 fill:#D5F5E3,stroke:#239B56,color:#000
    style D2 fill:#D5F5E3,stroke:#239B56,color:#000
    style D3 fill:#D5F5E3,stroke:#239B56,color:#000
    style E fill:#D6EAF8,stroke:#1F618D,stroke-width:2px,color:#000
    style F fill:#FDEDEC,stroke:#C0392B,stroke-width:2px,color:#000
    style G fill:#FEF9E7,stroke:#D4AC0D,stroke-width:2px,color:#000
    style H fill:#E8F8F5,stroke:#117864,stroke-width:3px,color:#000
```

---

## Diagram 3 — Simple Process Flow
> Use for: Slide 5 (if space is tight, most readable)

```mermaid
flowchart LR
    A(["Band 10\nGeoTIFF"]) --> B["Physics-Aware\nPreprocessing"]
    B --> C["Super-Resolution\nReal-ESRGAN"]
    B --> D["Semantic Masking\nSegFormer"]
    C & D --> E["Colorization\nAttention U-Net\nand PatchGAN"]
    E --> F["Post-Processing\nand Tile Stitching"]
    F --> G(["Trustworthy\nRGB Output"])
    G --> H["Evaluate\nPSNR, SSIM, LPIPS, FID"]

    style A fill:#dae8fc,stroke:#6c8ebf,color:#000
    style B fill:#fff2cc,stroke:#d6b656,color:#000
    style C fill:#d5e8d4,stroke:#82b366,color:#000
    style D fill:#e1d5e7,stroke:#9673a6,color:#000
    style E fill:#f8cecc,stroke:#b85450,color:#000
    style F fill:#ffe6cc,stroke:#d79b00,color:#000
    style G fill:#d5e8d4,stroke:#82b366,color:#000
    style H fill:#f5f5f5,stroke:#666,color:#000
```

---

## Diagram 4 — Colorization Model Detail
> Use for: Slide 7 (Architecture deep-dive)

```mermaid
flowchart TD
    IN1(["SR Thermal Patch\n1-channel"]) --> CAT
    IN2(["Semantic Mask\n4-channel\nWater, Veg, Urban, Soil"]) --> CAT
    CAT["Concatenate\n5-channel input"] --> GEN

    subgraph GEN["Attention U-Net Generator"]
        direction TB
        E1["Encoder Block 1\nConv + BN + LeakyReLU"]
        E2["Encoder Block 2"]
        E3["Encoder Block 3 - Bottleneck"]
        ATT["Attention Gates\non Skip Connections"]
        D3["Decoder Block 3"]
        D2["Decoder Block 2"]
        D1["Decoder Block 1\nConv + Tanh"]
        E1 --> E2 --> E3
        E3 --> D3
        E1 & E2 -.->|skip + attention| ATT
        ATT --> D3 & D2 & D1
        D3 --> D2 --> D1
    end

    D1 --> PRED(["Predicted RGB\n3-channel"])
    PRED --> DISC

    subgraph DISC["PatchGAN Discriminator"]
        DC["70x70 patch-level\nreal / fake classification"]
    end

    DISC -->|adversarial gradient| GEN

    GT(["Ground Truth\nBands 4-3-2"]) --> LOSS
    PRED --> LOSS

    subgraph LOSS["Composite Loss"]
        direction LR
        L1["L_GAN\nAdversarial"]
        L2["L_L1\nPixel-wise"]
        L3["L_VGG\nPerceptual"]
        L4["L_Sem\nSemantic"]
    end

    style GEN fill:#5c1a1a,color:#fff,stroke:#e74c3c
    style DISC fill:#1a3a5c,color:#fff,stroke:#4a90d9
    style LOSS fill:#3d3d00,color:#fff,stroke:#f39c12
```
