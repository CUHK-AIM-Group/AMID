# Seg-A 2023 Segmentation of the Aortic Vessel Tree

## Challenge and Evaluation
Seg-A 2023 is a 3D aortic vessel tree segmentation task. The submission contains one binary vessel-tree mask per case. Official evaluation reports DSC and Hausdorff distance on the 50% case subset; DSC is higher-better and HD is lower-better.

## Final Solution
### Method Overview
The final solution is a supervised 3D nnU-Net vessel segmentation pipeline followed by conservative anatomical cleanup. The model predicts the aortic vessel tree as a binary volume. Post-processing then removes disconnected islands and fills small holes, producing a contiguous vessel-tree mask for each case.

### Model Architecture
The model is a 3D full-resolution nnU-Net with a residual encoder. The residual encoder improves feature propagation through the 3D network, which is useful for thin vascular structures that extend over long distances. The decoder produces a binary vessel probability map at voxel level.

### Training Strategy
Training uses the standard nnU-Net automatic planning workflow for 3D medical segmentation. The model is trained on the 35 visible labeled cases for 100 epochs with the standard nnU-Net supervised objective. The best trained residual-encoder model from this training pass is used for final inference.

### Inference Strategy
Inference produces one volumetric vessel mask for each of the 12 test cases. Test-time augmentation is disabled to keep the output deterministic and stable. Very small disconnected components below 300 voxels are removed during export before the final cleanup step.

### Post-processing
Final post-processing keeps the largest 3D connected vessel component and fills internal binary holes. This favors one anatomically coherent aortic vessel tree rather than many disconnected fragments. Per-case cleanup statistics confirm how many components and voxels were removed before final submission.

## Internal Validation
Internal records report a best one-fold score of 0.9197972879 for the source model. A stronger reference score of 0.9352773889 was available from top-method comparison, but the final submitted solution uses the described residual-encoder nnU-Net with complete cleanup statistics for all 12 test cases.

## Official Test Result
- Leaderboard score / mean position: `9.5`
- Medal: none
- DSC 50pc: `0.91426`
- HD 50pc: `7.59674`
- Percentile: `0.15`
