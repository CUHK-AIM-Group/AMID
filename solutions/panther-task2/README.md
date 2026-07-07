# PANTHER Task 2 Pancreatic Tumor Segmentation on MR-Linac MRI

## Challenge and Evaluation
PANTHER Task 2 is binary pancreatic tumor segmentation on MR-Linac MRI. The submission contains one 3D tumor mask per case. Official evaluation includes DSC and several distance or volume error metrics; DSC is higher-better, while MSD, HD95, MASD, and RMSE are lower-better.

## Final Solution
### Method Overview
The final solution is a five-fold 3D SegResNet ensemble for pancreatic tumor segmentation on MR-Linac MRI. The method focuses the model on a pancreas-centered region of interest, predicts a tumor mask inside that crop, then restores the prediction to the original 3D image geometry. The five fold predictions are averaged uniformly for the final submission.

### Model Architecture
The segmentation network is a compact 3D MONAI SegResNet. It uses residual convolutional blocks in an encoder-decoder layout and outputs a sigmoid tumor probability map. The initial channel width is 8 filters, with a shallow residual block layout chosen to keep the model small enough for repeated 3D fold training.

Preprocessing rescales intensities using robust nonzero percentiles, crops around the estimated pancreatic region, and resizes the crop to 64 by 128 by 128 voxels. This gives the model a consistent field of view while preserving enough local context around the tumor.

### Training Strategy
Each fold is trained for 24 epochs with AdamW, learning rate 3e-4, and weight decay 1e-5. Random axis flips, mild intensity augmentation, and 0.05 dropout are used to improve robustness to MR-Linac appearance variation. The loss combines Dice loss with a smaller binary cross-entropy term, and the positive class is upweighted to compensate for the small tumor volume relative to background.

### Inference Strategy
Inference uses the same ROI definition and resizing as training. Each fold predicts a tumor probability map inside the crop, applies a fold-level threshold of 0.60, and pastes the resulting mask back into the original case geometry. The five restored masks are then averaged and thresholded at 0.5 to produce the submitted binary mask.

### Post-processing
Post-processing removes scattered false positives by retaining the dominant connected component. Two light binary erosion passes are applied before the final component cleanup, making the output more conservative and reducing small disconnected islands. The final mask is the uniform ensemble of the cleaned fold predictions.

## Internal Validation
Internal records confirm that five fold models were trained and used for final inference. Training and inference logs are available for the fold predictions, but no standalone numeric cross-validation summary was preserved for this final submission.

## Official Test Result
- Leaderboard score / mean position: `9.6`
- Medal: none
- DSC: `0.30851`
- MSD: `0.38614`
- HD95: `93.02785`
- MASD: `75.9348`
- RMSE: `28452.84816`
- Percentile: `0.14`
