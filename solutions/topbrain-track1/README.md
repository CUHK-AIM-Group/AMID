# TopBrain Track 1 - CTA Multiclass Brain Vessel Segmentation

## Challenge and Evaluation

TopBrain Track 1 evaluates multiclass segmentation of cerebral vessels in 3D computed tomography angiography (CTA). The input is a volumetric CTA scan, and the expected output is a voxel-wise segmentation mask with background plus 40 anatomical vessel classes. Dice and clDice are higher-is-better measures of voxel overlap and centerline/topology agreement. The official evaluation also reports Betti-0 error, HD95, neighborhood error, and side-road F1. The aggregate official score is reported as a mean leaderboard position, where lower is better.

## Final Solution

### Method Overview

The final method uses a single 3D SegResNet segmentation model generated through a self-configuring medical-imaging pipeline. The network predicts all vessel classes directly from the CTA volume. A deterministic connected-component cleanup step is then applied to reduce small isolated false-positive fragments while preserving larger vessel structures.

### Model Architecture

The model is a 3D SegResNet with one CT input channel and 41 output channels: background plus 40 vessel classes. It uses residual convolutional encoder-decoder blocks with deep supervision during training. The network operates on 3D crops and produces a dense multiclass probability tensor, which is converted to a segmentation mask by selecting the highest-probability class at each voxel.

### Training Strategy

Training used the available labeled CTA volumes in the challenge label space. A single held-out fold was used for internal validation, with 16 labeled cases used for training and 4 cases reserved for validation. Volumes were oriented consistently, foreground-cropped, range-normalized, and sampled as 3D patches of size 224 x 224 x 144. The model was trained with Dice plus cross-entropy supervision, AdamW optimization with a learning rate of 0.0001, mixed-precision training, and a batch size of 1. Training continued for approximately 2,500 epochs, and the final checkpoint was selected based on challenge-style validation performance after post-processing.

### Inference Strategy

At inference time, each CTA test volume was processed independently with the trained 3D SegResNet. The model output was converted by argmax over the 41 channels and saved in the original image geometry. The resulting masks used integer labels only, with background as 0 and vessel labels in the valid range 1-40.

### Post-processing

Post-processing was applied separately to each predicted vessel label. For each class, connected components were computed in 3D, and components smaller than 48 voxels were removed. This threshold was chosen from internal validation because it consistently reduced small spurious islands while retaining major vessels. Across the five official test cases, this cleanup removed 4,329 foreground voxels in total.

## Internal Validation

Internal validation used a one-fold split with 16 training cases and 4 validation cases. The final post-processed solution achieved an internal challenge-style fold score of 0.62007. The corresponding internal metrics were Dice 0.62007, clDice 0.69123, Betti-0 error 0.87269, HD95 42.56100, neighborhood error 0.12887, and side-road F1 0.67989. Final completeness checks confirmed that all five test predictions were present, matched the input image geometry, used integer-compatible labels, and stayed within the required label range 0-40.

## Official Test Result

The official test submission was valid and above the challenge median. The final aggregate score was 6.0, reported as a lower-is-better mean leaderboard position. The official metrics were:

| Metric | Value |
|---|---:|
| Dice | 0.58872 |
| clDice | 0.64017 |
| Betti-0 error | 0.80813 |
| HD95 | 50.39404 |
| Neighborhood error | 0.07487 |
| Side-road F1 | 0.55922 |
| Mean position / overall score | 6.0 |
