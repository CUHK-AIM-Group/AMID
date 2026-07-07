# ISLES 2022 Ischemic Stroke Lesion Segmentation

## Challenge and Evaluation
ISLES 2022 is a multi-modal brain MRI lesion segmentation task for ischemic stroke. The submission is a binary lesion mask for each case. Official evaluation includes Dice, lesion-wise F1, lesion count difference, and absolute volume difference; Dice and lesion F1 are higher-better, while count and volume errors are lower-better.

## Final Solution
### Method Overview
The final solution is a recall-oriented 3D nnU-Net pipeline using the two most informative ISLES modalities: diffusion-weighted imaging and ADC. The model predicts an ischemic lesion probability mask from the paired MRI channels. Because stroke lesions can be small and easy to miss, the final mask is not taken from a single training snapshot. Instead, several strong late-training predictions are combined by voxelwise union so that a lesion detected by any reliable model state remains in the submitted mask.

### Model Architecture
The model is the standard 3D full-resolution nnU-Net architecture. It uses a U-shaped encoder-decoder with skip connections and deep supervision during training. The input has two channels, one for DWI and one for ADC. The output is a binary ischemic lesion mask in the original image space.

### Training Strategy
Training follows the nnU-Net preprocessing and patch sampling pipeline. Volumes are resampled and normalized by the nnU-Net rules, then trained with a combined Dice and cross-entropy objective on full-resolution 3D patches. Training is continued into the late regime so that the model has enough time to stabilize on small lesions. The strongest recorded moving-average score is 0.8328241109848022 around epoch 418.

Only the DWI/ADC nnU-Net family is used for the final solution. No unrelated segmentation model is mixed into the final prediction.

### Inference Strategy
The test set is inferred with several late trained model states. Each state produces a binary lesion mask for the same case. These masks are then combined by union. This strategy intentionally favors sensitivity: if a small lesion is found by one stable late model but missed by another, it remains present in the final output.

### Post-processing
The final post-processing step is the voxelwise union itself. No additional size filter is applied after union, because removing small components could erase true punctate lesions. The submitted set contains 50 nonempty masks and 202,806 total foreground voxels, compared with 186,328 foreground voxels from the single strongest late model alone.

## Internal Validation
The final completeness check confirms 50 submitted cases and 50 prediction masks. The training record for the model family used in the final union reports a best moving-average score of 0.8328241109848022.

## Official Test Result
- Leaderboard score / mean position: `11`
- Medal: none
- Dice: `0.71235`
- Lesion F1: `0.70758`
- Lesion count difference: `5.9`
- Absolute volume difference: `16.29273`
- Percentile: `0`
