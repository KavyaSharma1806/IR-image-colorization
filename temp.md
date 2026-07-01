# SMART-VIS — PPT-Ready Mermaid Diagrams
> Paste at **https://mermaid.live** → Export PNG (1920x1080) → Insert into slide

---

## SLIDE 5 — Process Flow Diagram
> Clean left-to-right, fits perfectly on one landscape slide

```mermaid
flowchart LR
    A(["🛰️ Landsat-9\nBand 10\nGeoTIFF"])

    subgraph PRE["⚙️ Preprocessing"]
        B["DN to\nBrightness Temp\nCloud Masking"]
    end

    subgraph ENH["🔬 Enhancement"]
        C["Super-Resolution\nReal-ESRGAN"]
    end

    subgraph SEM["🗺️ Semantic Guidance"]
        D["Land-Cover Masking\nSegFormer"]
    end

    subgraph COL["🎨 Colorization"]
        E["Attention U-Net\nPix2Pix cGAN"]
    end

    subgraph REF["🧹 Refinement"]
        F["Histogram Match\nTile Stitching"]
    end

    G(["🖼️ RGB\nGeoTIFF\nOutput"])

    A --> PRE --> ENH --> SEM --> COL --> REF --> G

    style PRE fill:#1a3a5c,color:#fff,stroke:#4a90d9,stroke-width:2px
    style ENH fill:#1a4731,color:#fff,stroke:#27ae60,stroke-width:2px
    style SEM fill:#3d1a5c,color:#fff,stroke:#8e44ad,stroke-width:2px
    style COL fill:#5c1a1a,color:#fff,stroke:#e74c3c,stroke-width:2px
    style REF fill:#5c4200,color:#fff,stroke:#e67e22,stroke-width:2px
    style A fill:#2c3e50,color:#fff,stroke:#95a5a6,stroke-width:2px
    style G fill:#1a4731,color:#fff,stroke:#27ae60,stroke-width:3px
```

---

## SLIDE 5 (Alt) — Process Flow with Parallel Branch
> Shows SR + Semantic happening in parallel — slightly more technical

```mermaid
flowchart LR
    A(["🛰️ Band 10\nGeoTIFF"]) --> B

    B["⚙️ Physics-Aware\nPreprocessing\nDN to Brightness Temp"]

    B --> C["🔬 Super-Resolution\nReal-ESRGAN\n30m equivalent"]
    B --> D["🗺️ Semantic Masking\nSegFormer\nWater, Veg, Urban, Soil"]

    C --> E
    D --> E

    E["🎨 Colorization\nAttention U-Net + PatchGAN\nGAN + L1 + VGG + Semantic Loss"]

    E --> F["🧹 Post-Processing\nHistogram Match + Tile Stitch"]

    F --> G(["🖼️ Trustworthy RGB\nFull-Scene GeoTIFF"])

    G --> H["📊 PSNR · SSIM\nLPIPS · FID"]

    style A fill:#2980b9,color:#fff,stroke:#1a5276,stroke-width:2px
    style B fill:#8e44ad,color:#fff,stroke:#6c3483,stroke-width:2px
    style C fill:#27ae60,color:#fff,stroke:#1e8449,stroke-width:2px
    style D fill:#16a085,color:#fff,stroke:#0e6655,stroke-width:2px
    style E fill:#c0392b,color:#fff,stroke:#922b21,stroke-width:2px
    style F fill:#d35400,color:#fff,stroke:#a04000,stroke-width:2px
    style G fill:#27ae60,color:#fff,stroke:#1e8449,stroke-width:3px
    style H fill:#2c3e50,color:#fff,stroke:#95a5a6,stroke-width:2px
```

---

## SLIDE 7 — Architecture Diagram
> Top-down system architecture, fits a full slide cleanly

```mermaid
flowchart TD
    IN(["🛰️ INPUT\nLandsat-9 Band 10\nRaw GeoTIFF"])

    subgraph PIPE["SMART-VIS Pipeline"]
        direction TB

        P["⚙️ Physics Layer\nDN → Brightness Temp · QA Cloud Mask · Patch Extraction"]

        P --> SR["🔬 Real-ESRGAN\nSuper-Resolution\n100m → 30m equivalent"]
        P --> SEG["🗺️ SegFormer\nLand-Cover Segmentation\nWater · Veg · Urban · Soil"]

        SR --> FUSE["🔗 Feature Fusion\n5-channel Input\n1ch Thermal + 4ch Mask"]
        SEG --> FUSE

        FUSE --> GAN["🎨 Pix2Pix cGAN\nAttention U-Net Generator\nPatchGAN Discriminator\nLoss: GAN + L1 + VGG + Semantic"]

        GAN --> POST["🧹 Post-Processing\nHistogram Matching · Tile Stitching · Geo-metadata Restore"]
    end

    OUT(["🖼️ OUTPUT\nHigh-Fidelity RGB GeoTIFF\nSemantically Accurate · Geospatially Correct"])
    EVAL["📊 PSNR · SSIM · LPIPS · FID · Inference Time"]

    IN --> PIPE
    PIPE --> OUT
    OUT --> EVAL

    style IN fill:#2980b9,color:#fff,stroke:#1a5276,stroke-width:2px
    style P fill:#8e44ad,color:#fff,stroke:#6c3483,stroke-width:2px
    style SR fill:#27ae60,color:#fff,stroke:#1e8449,stroke-width:2px
    style SEG fill:#16a085,color:#fff,stroke:#0e6655,stroke-width:2px
    style FUSE fill:#2c3e50,color:#fff,stroke:#95a5a6,stroke-width:2px
    style GAN fill:#c0392b,color:#fff,stroke:#922b21,stroke-width:2px
    style POST fill:#d35400,color:#fff,stroke:#a04000,stroke-width:2px
    style OUT fill:#27ae60,color:#fff,stroke:#1e8449,stroke-width:3px
    style EVAL fill:#2c3e50,color:#fff,stroke:#95a5a6,stroke-width:2px
    style PIPE fill:#1a1a2e,color:#fff,stroke:#4a90d9,stroke-width:2px
```

---

## Export Tips for PPT
1. Go to https://mermaid.live
2. Paste the diagram code → it renders on the right
3. Click **"Export"** → choose **PNG** → set width to **1920px**
4. Insert the PNG image into your PPT slide
5. Set slide background to **dark (#0d1117)** for best contrast with the colored nodes
