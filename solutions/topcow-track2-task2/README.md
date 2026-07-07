# TopCoW Track 2 Task 2 - MRA 3D Bounding Box Detection

## Challenge and Evaluation

This challenge evaluates localization of the Circle of Willis region in 3D magnetic resonance angiography volumes. Each input is a 3D MRA NIfTI image, and the expected output is one 3D bounding box per case. The submitted prediction for each case is a JSON object containing an integer voxel-space box size and an integer voxel-space minimum-corner location.

The primary image-analysis objective is accurate 3D spatial overlap between the predicted and reference bounding boxes. The key metrics are standard 3D IoU and boundary IoU; higher IoU values indicate better localization. The official aggregate challenge score is reported as a rank-style score where lower is better.

## Final Solution

### Method Overview

The final solution is a deterministic ensemble of complementary 3D bounding-box predictors. The strongest component is a segmentation-derived box estimator trained to predict a coarse Circle of Willis region mask and then convert that mask into a bounding box. A second component is a compact direct 3D detector inspired by Retina U-Net, used to provide an independent localization estimate. A third segmentation-derived estimate is used as a robustness correction for the final public-test export.

The final box is produced directly in the original image voxel coordinate system. The ensemble combines predicted minimum and maximum box corners, rather than averaging only centers and sizes, so that the final integer box remains geometrically consistent after rounding.

### Model Architecture

The main segmentation branch uses a 3D SegResNet-style encoder-decoder trained on rasterized bounding-box masks. It predicts a coarse foreground probability map for the Circle of Willis region. The probability map is thresholded, the dominant connected component is selected, and the component center is converted into a bounding box using a learned size prior from the training boxes.

The direct detection branch is a compact one-box 3D detector following the Retina U-Net family of dense medical detection models. It predicts a task-level box estimate without first producing a segmentation mask. This branch is used with a small weight in the final ensemble because it provides a complementary correction to the segmentation-derived boxes.

The final robustness branch is another SegResNet-derived box extractor. It uses the same coarse-mask principle but extracts centers with a weighted largest-component rule and a median-size prior. This branch was included to stabilize the public-test export against small center shifts.

### Training Strategy

Training used the available labeled MRA training cases with locked cross-validation folds. The segmentation-derived box models were trained as 3D mask predictors from bounding-box supervision converted into binary box masks. Training used small 3D patches to fit the full-resolution anisotropic volumes in memory, Dice-style mask optimization, and multiple random seeds for the main segmentation model.

For the main SegResNet box branch, three seed models were trained for 40 epochs with 3D patches of 64 x 80 x 32 voxels. The foreground probability threshold for the main mask-to-box conversion was 0.65, with largest-component filtering and a size prior based on the 55th percentile of training-box sizes. The robustness segmentation branch used a threshold of 0.56, weighted component centers, and a median-size prior. The direct detector was trained and checkpointed separately on the same locked-fold protocol, then used only as a lightweight correction signal in the final ensemble.

### Inference Strategy

At inference time, each public-test MRA volume is processed independently. The segmentation branches generate foreground probability maps and convert them into voxel-space boxes. The direct detector generates a second voxel-space box estimate. All outputs are represented as minimum and maximum corners before ensembling.

The first ensemble combines the main segmentation-derived box with the direct detector box. The direct detector contributes 20% of the corner coordinates, while the segmentation-derived box contributes 80%. The final public-test export then blends this anchor box with the robustness segmentation box; the robustness box contributes 25% of the final corner coordinates. After blending, both corners are rounded to integer voxel coordinates and the final box size is recomputed from the rounded corners.

### Post-processing

Post-processing is intentionally simple and deterministic. Segmentation probability maps are thresholded, only the largest connected component is retained, and small spurious regions are discarded by this component selection. Box sizes are constrained through training-set size priors rather than raw mask extents alone. The final ensemble operates in original image voxel coordinates, rounds blended corners to integers, and enforces positive box sizes for every case.

## Internal Validation

Internal validation used a locked five-fold cross-validation protocol over the labeled training set, with 80 training cases and 20 validation cases per fold. The final method family was evaluated on all five folds before being used for the public-test export.

The main segmentation-derived component reached a five-fold mean IoU of 0.7767. The direct detector component reached a five-fold mean IoU of 0.7588. Blending the two components by voxel-space corners improved the internal five-fold mean IoU to 0.7804. A separate segmentation-derived robustness branch reached a five-fold mean IoU of 0.7779 and passed the same fold-completeness and leakage checks. The final public-test export included all 25 required test cases, with one valid JSON bounding-box prediction for each case and row order matching the sample submission.

## Official Test Result

The official public-test submission was valid. The official aggregate score was 4.5, where lower is better for the aggregate rank-style score.

Key official metrics:

| Metric | Value |
|---|---:|
| Boundary IoU | 0.74131 |
| 3D IoU | 0.75599 |
| Boundary IoU position | 2.0 |
| IoU position | 7.0 |
| Mean position / overall score | 4.5 |
| Percentile | 0.5 |

The submission did not meet the medal or above-median thresholds under the official aggregate ranking criteria.
