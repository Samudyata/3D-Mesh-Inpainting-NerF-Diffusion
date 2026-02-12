<p align="center">
  <h1 align="center">Hybrid Inpainting for 3D Meshes</h1>
  <p align="center">
    <strong>NeRF + Diffusion + Custom PCN for Watertight 3D Mesh Reconstruction</strong>
  </p>
  <p align="center">
    <a href="https://www.python.org/"><img src="https://img.shields.io/badge/Python-3.8+-3776AB?logo=python&logoColor=white" alt="Python"></a>
    <a href="https://pytorch.org/"><img src="https://img.shields.io/badge/PyTorch-2.0+-EE4C2C?logo=pytorch&logoColor=white" alt="PyTorch"></a>
    <a href="http://www.open3d.org/"><img src="https://img.shields.io/badge/Open3D-0.17+-4B8BBE?logo=open3d&logoColor=white" alt="Open3D"></a>
    <a href="https://trimesh.org/"><img src="https://img.shields.io/badge/Trimesh-3.x-orange" alt="Trimesh"></a>
    <a href="https://numpy.org/"><img src="https://img.shields.io/badge/NumPy-1.24+-013243?logo=numpy&logoColor=white" alt="NumPy"></a>
    <a href="https://objaverse.allenai.org/"><img src="https://img.shields.io/badge/Objaverse-Dataset-green" alt="Objaverse"></a>
  </p>
</p>

---

## Overview

3D mesh data captured from real-world sources is often incomplete due to sensor occlusions, surface reflectivity, or limited scanning angles. This project presents a **hybrid reconstruction pipeline** that combines classical geometric methods, a custom learning-based point completion model, and neural radiance field predictions to reconstruct watertight 3D meshes from partial inputs.

This work was completed as an academic research project at Arizona State University.

---

## Pipeline Overview

The hybrid approach operates in four stages:

```
Input (Partial Mesh)
        |
        v
[Stage 1] Dataset Creation
        |   - Download meshes from Objaverse (LVIS subset)
        |   - Programmatically introduce holes to simulate real-world damage
        |   - Generate paired (incomplete, ground-truth) training data
        v
[Stage 2] Custom PCN Training
        |   - Encoder: 1D convolution layers -> global feature (1024-d)
        |   - Coarse output via fully connected layers
        |   - FoldingNet decoder with 2D grid + skip connections for fine output
        |   - Loss: Chamfer Distance + Repulsion Loss + Structure Loss
        |   - Trained for 50 epochs on cached point cloud pairs
        v
[Stage 3] NeRFiller Multi-View Inpainting
        |   - Render RGB + depth views of the partial mesh
        |   - Use NeRFiller (diffusion-based) to inpaint missing regions
        |   - Back-project inpainted depth maps to 3D point clouds
        v
[Stage 4] Fusion and Surface Reconstruction
            - Align NeRF-generated point cloud with the partial mesh (ICP)
            - Fuse point clouds and remove statistical outliers
            - Reconstruct surface via Poisson Reconstruction
            - Refine with Ball Pivoting Algorithm (BPA) for watertight output
```

---

## Quick Start

### Prerequisites

- Python 3.8+
- CUDA-capable GPU (recommended for PCN training)
- [NeRFiller](https://github.com/ethanweber/nerfiller) installed separately (used in Stage 3)

### Installation

```bash
# Clone the repository
git clone https://github.com/Samudyata/3D-Mesh-Inpainting-NeRF-Diffusion.git
cd 3D-Mesh-Inpainting-NeRF-Diffusion

# Install dependencies
pip install -r requirements.txt
```

### Usage

1. **Generate training data** -- Create partial meshes with synthetic holes from Objaverse models:
   ```bash
   python datset_creation.py
   ```

2. **Train the custom PCN** -- Train the Point Completion Network on paired (incomplete, complete) point clouds:
   ```bash
   python custompcn.py
   ```
   > Note: `custompcn.py` was originally developed in Google Colab. Some shell commands (lines starting with `!`) may need to be run manually or removed before executing locally.

3. **Run NeRFiller** -- Obtain multi-view inpainted renders using the [NeRFiller repository](https://github.com/ethanweber/nerfiller). Follow their setup instructions separately.

4. **Fuse and reconstruct** -- Open `Hybrid_Model.ipynb` in Jupyter to align NeRF outputs with partial meshes, fuse point clouds, and perform Poisson + BPA surface reconstruction.

---

## Project Structure

```
.
|-- README.md                                           # Project documentation
|-- requirements.txt                                    # Python dependencies
|-- .gitignore                                          # Git ignore rules
|
|-- custompcn.py                                        # Custom PCN model: encoder, FoldingNet
|                                                       #   decoder, training loop, and visualization
|                                                       #   (exported from Colab)
|
|-- datset_creation.py                                  # Dataset preparation script: downloads
|                                                       #   Objaverse meshes and creates holed
|                                                       #   variants for training (exported from Colab)
|
|-- Hybrid_Model.ipynb                                  # Main notebook for the full hybrid pipeline:
|                                                       #   depth-to-pointcloud conversion, ICP
|                                                       #   alignment, point cloud fusion, Poisson
|                                                       #   reconstruction, and Ball Pivoting
|
|-- Hybrid Inpainting of 3D Meshes_Final Report.pdf    # Full technical report
|-- Hybrid Inpainting of 3D mesh_Samudyata_ppt.pdf     # Presentation slides
```

---

## Tech Stack

| Component            | Technology                                                   |
|----------------------|--------------------------------------------------------------|
| Deep Learning        | PyTorch (Conv1d encoder, FoldingNet decoder, Chamfer loss)   |
| 3D Geometry          | Open3D (Poisson reconstruction, BPA, ICP alignment)          |
| Mesh Processing      | Trimesh (mesh loading, sampling, hole creation)              |
| NeRF Inpainting      | [NeRFiller](https://github.com/ethanweber/nerfiller) (external) |
| Dataset              | [Objaverse](https://objaverse.allenai.org/) (LVIS subset)   |
| Numerical Computing  | NumPy, Matplotlib                                            |
| Word Embeddings      | Gensim (for semantic category filtering)                     |

---

## Results

Quantitative evaluation on NeRFiller-rendered views and hybrid reconstructions:

| Mesh    | Method         | PSNR (dB) | SSIM  | Notes                                        |
|---------|----------------|----------:|------:|----------------------------------------------|
| Chair   | Nerfacto       |      6.95 | 0.733 | Baseline NeRF without inpainting             |
| Chair   | Grid-Prior     |     21.42 | 0.879 | NeRFiller with grid-prior diffusion guidance  |
| Box     | Grid-Prior     |     31.57 | 0.974 | Strong performance on simple geometry         |
| Trawler | Grid-Prior     |     33.81 | 0.978 | Best quantitative result                      |
| Boot    | Hybrid (Ours)  |        -- |    -- | Reconstructed with high detail and structure  |

> PSNR and SSIM are computed between inpainted NeRF renders and ground-truth views. The hybrid method (Boot) was evaluated qualitatively on mesh completeness and watertightness.

---

## Future Work

- Improve NeRF-to-mesh alignment using learned registration networks
- Add semantic or class-aware priors to guide the inpainting process
- Evaluate on a broader range of Objaverse categories and real scanned data
- Replace Colab shell commands in scripts with proper Python equivalents

---

## Contributors

- **Samudyata Jagirdar** (`sjagird1@asu.edu`)
- Shantanu Patne
- Amogh Ravindra Rao
- Kristy McWilliams
- Surgit Madhavhen

---

## Acknowledgements

- [NeRFiller](https://github.com/ethanweber/nerfiller) -- Weber et al., for diffusion-based multi-view inpainting
- [PCN-PyTorch](https://github.com/qinglew/PCN-PyTorch) -- Reference implementation for Point Completion Networks
- [Objaverse](https://objaverse.allenai.org/) -- Large-scale 3D object dataset

---

## License

This project is for academic and research purposes. Please see individual dependencies for their respective licenses.
