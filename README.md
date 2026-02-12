# 🧠 Hybrid Inpainting for 3D Meshes

This repository presents a hybrid 3D mesh inpainting pipeline that integrates classical geometric methods, a custom learning-based point completion model, and radiance field predictions to reconstruct incomplete 3D meshes.

---

## 🚀 Project Summary

3D mesh data from real-world sources is often incomplete due to occlusions, sensor limitations, or reflectivity issues. This project proposes a hybrid reconstruction strategy that:

- Uses geometric methods like Poisson surface reconstruction.
- Employs a custom implementation of the **Point Completion Network (PCN)**.
- Integrates rendered views from **NeRFiller** for multi-view RGB and depth inpainting.
- Fuses visual and geometric data using **Ball Pivoting Algorithm (BPA)** to create watertight meshes.

---

## 📁 Files Included

- `customPCN.ipynb` – Custom PCN architecture (FoldingNet-based refinement)
- `dataset_creation.py` – Script to prepare partial (holed) mesh datasets
- `Hybrid_Model.ipynb` – Main notebook for NeRF + PCN fusion and reconstruction
- `Hybrid Inpainting of 3D Meshes_Final Report.pdf` – Full technical report
- `Hybrid Inpainting of 3D mesh_Samudyata_ppt.pdf` – Presentation slides
- `README.md` – This documentation

📌 Note: The NeRFiller component used in our pipeline is based on:
[https://github.com/ethanweber/nerfiller](https://github.com/ethanweber/nerfiller)

We do not include that source code here. Please download and configure it separately if you wish to reproduce our results.

---

## 📊 Sample Results

| Mesh     | Method     | PSNR (↑) | SSIM (↑) |
|----------|------------|----------|----------|
| Chair    | Nerfacto   | 6.95     | 0.733    |
| Chair    | Grid-Prior | 21.42    | 0.879    |
| Box      | Grid-Prior | 31.57    | 0.974    |
| Trawler  | Grid-Prior | 33.81    | 0.978    |
| Boot     | Hybrid     | ✅ Reconstructed with high detail + structure |

---

## 🛠 How to Use

1. Generate holed datasets using `dataset_creation.py`.
2. Train the custom PCN model using `customPCN.ipynb`.
3. Obtain NeRF inpainted renders using the original [NeRFiller repo](https://github.com/ethanweber/nerfiller).
4. Use `Hybrid_Model.ipynb` to fuse outputs and reconstruct final meshes.

---

## 📌 Future Improvements

- Enhance NeRF-mesh alignment with learned registration
- Add class/semantic awareness to the inpainting task
- Evaluate on wider Objaverse categories

---

## 👨‍💻 Contributors

- Samudyata Jagirdar (`sjagird1@asu.edu`)
- Shantanu Patne
- Amogh Ravindra Rao
- Kristy McWilliams
- Surgit Madhavhen

---

## 🙌 Acknowledgements

- [NeRFiller](https://github.com/ethanweber/nerfiller)
- [PCN-PyTorch](https://github.com/qinglew/PCN-PyTorch)
- [Objaverse Dataset](https://github.com/OpenRobotLab/objaverse)


