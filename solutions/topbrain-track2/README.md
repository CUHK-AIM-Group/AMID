# TopBrain Track 2 - MRA Multiclass Brain Vessel Segmentation

## Challenge and Evaluation

TopBrain Track 2 evaluates multiclass segmentation of cerebral vessels in 3D magnetic resonance angiography (MRA). The input is a volumetric MRA scan, and the expected output is a voxel-wise segmentation mask with background plus 42 anatomical vessel classes. The primary segmentation metric is Dice, where higher values indicate better class overlap. The official evaluation also reports topology-aware and boundary-related measures, including clDice, Betti-0 error, HD95, neighborhood error, and side-road F1. The final aggregate score is reported as a mean leaderboard position, where lower is better.

## Final Solution

### Method Overview

The final method uses a single full-resolution 3D U-Net-style segmentation model followed by connected-component cleanup. The neural network produces the initial 42-class vessel mask directly from the MRA volume. A lightweight topology-oriented post-processing step then removes very small disconnected per-class islands, which are more likely to be false-positive fragments than true vessel branches.

### Model Architecture

The segmentation backbone is a self-configuring 3D U-Net operating at full image resolution. It uses an encoder-decoder design with six resolution stages, 3D convolutions, instance normalization, leaky-ReLU activations, and skip connections between matched encoder and decoder stages. The model predicts all vessel labels jointly as a 43-class segmentation problem, including background.

### Training Strategy

The model was trained on the labeled MRA training volumes using the challenge label space directly. Images were normalized with z-score intensity normalization and resampled according to the automatically selected full-resolution 3D plan. Training used patch-based 3D sampling with a patch size chosen to fit the anisotropic MRA volumes, batch Dice plus cross-entropy style supervision, and standard nnU-Net-style data augmentation. The selected checkpoint was trained for 250 epochs and chosen using internal validation behavior rather than the external test labels.

### Inference Strategy

At inference time, each test MRA volume was processed with sliding-window 3D prediction at full resolution. Mirroring-based test-time augmentation was enabled. The model output was resampled back into the original image geometry, and the final prediction was saved as an integer-valued multiclass NIfTI mask with labels restricted to the valid range 0-42.

### Post-processing

Post-processing was applied independently to each non-background vessel class. For each label, 6-connected components were computed in 3D. Components smaller than 75 voxels were removed, while larger components were retained unchanged. This cleanup removed small isolated fragments while preserving the main vessel structures and all remaining labels. On the official test cases, this step removed 6,544 foreground voxels in total across the five submitted masks.

## Internal Validation

Internal validation used a held-out fold from the available MRA training set: 16 labeled cases were used for model fitting and 4 cases were used for fold-level validation. The selected 250-epoch model reached a visible validation mean Dice of 0.575 before post-processing. Submission completeness checks confirmed that all five test cases were present, each output matched the corresponding input image shape and spacing, masks used unsigned 8-bit integer labels, and all predicted labels were within the required 0-42 range.

## Official Test Result

The official test submission was valid. The final aggregate score was 9.5, reported as a lower-is-better mean leaderboard position. The official metrics were:

| Metric | Value |
|---|---:|
| Dice | 0.63812 |
| clDice | 0.67101 |
| Betti-0 error | 1.14211 |
| HD95 | 49.38801 |
| Neighborhood error | 0.34307 |
| Side-road F1 | 0.70420 |
| Mean position / overall score | 9.5 |
