# PANTHER Task 1 Pancreatic Tumor Segmentation on Diagnostic MRI

## Challenge and Evaluation
PANTHER Task 1 is binary pancreatic tumor segmentation on diagnostic MRI. The submission contains one 3D tumor mask per case. Official evaluation reports DSC and surface/volume error metrics such as MSD, HD95, MASD, and RMSE; DSC is higher-better, while distance and error metrics are lower-better.

## Final Solution
### Method Overview
The final solution is a five-fold 3D transformer segmentation ensemble for diagnostic MRI. Each model predicts a pancreatic tumor probability map from a cropped 3D MRI region, and the final mask is obtained by averaging the five fold predictions. This ensemble design reduces dependence on any single training split and makes the binary tumor output more stable.

### Model Architecture
The network is a compact 3D SegFormer-style model. A transformer encoder captures long-range 3D context around the pancreas, while a lightweight decoder converts the encoded features into a voxelwise tumor probability map. Training uses a Dice plus cross-entropy segmentation objective with an auxiliary loss to help intermediate features learn tumor localization. The model sees 3D patches centered near foreground regions, with random offsets so that it learns both tumor appearance and surrounding anatomy.

### Training Strategy
Training is organized as a gradual refinement schedule. The first stage learns the main tumor-localization behavior with learning rate 3e-4 for 1,200 steps. The second stage continues from the best first-stage model for 800 steps at learning rate 1e-4. The third stage performs a final 600-step refinement at learning rate 5e-5. Each stage uses a short warmup period to avoid unstable early updates.

Optimization uses AdamW, weight decay 5e-5, gradient clipping at 1.0, and batch size 1. Five models are trained on different cross-validation folds and are kept as equal contributors to the final ensemble.

### Inference Strategy
At inference time, MRI intensities are normalized with a foreground-based z-score rule. The model is applied with overlapping sliding windows so that full 3D cases can be predicted despite memory limits. Flip test-time augmentation is used along the three spatial axes, and the augmented probability maps are averaged before thresholding. Each fold produces a tumor probability map, and voxels above 0.5 are considered tumor for that fold.

### Post-processing
Each fold prediction is cleaned by keeping the largest connected tumor component, which removes isolated false-positive islands. The five cleaned probability maps are then averaged uniformly, and the final binary tumor mask is exported from the ensemble average.

## Internal Validation
Internal records confirm that all five folds were trained and used in the final ensemble, with prediction statistics available for the fold outputs and the final averaged masks. A standalone numeric cross-validation summary was not preserved for this final submission.

## Official Test Result
- Leaderboard score / mean position: `9.4`
- Medal: none
- DSC: `0.40832`
- MSD: `0.57393`
- HD95: `26.21856`
- MASD: `16.78252`
- RMSE: `18926.57507`
- Percentile: `0.16`
