# TopCoW Track 1 Task 1: CTA Multi-Class Segmentation

## Challenge and Evaluation

This challenge evaluates multi-class segmentation of the Circle of Willis and major intracranial arteries in CTA volumes. The input is a 3D CTA image, and the expected output is a voxel-wise segmentation mask containing background plus the TopCoW arterial labels, including the non-contiguous label 15.

The official evaluation combines overlap, centerline, boundary, and topology-sensitive metrics. Dice and clDice are higher-is-better component metrics, while HD95 and error metrics are lower-is-better. The final official score is reported as an overall mean ranking position, where lower is better.

## Final Solution

### Method Overview

The final submission used a single 3D nnU-Net-style residual encoder segmentation model trained for CTA vessel segmentation. The model produced full-volume multi-class hard masks for all test CTA cases. No multi-fold ensemble or probability averaging was used in the final submission.

### Model Architecture

The model was a 3D residual encoder U-Net with instance normalization and leaky ReLU activations. It used planner-selected CTA preprocessing, CT intensity normalization, and a full-resolution 3D setup with patch-based inference. The network predicted the Circle of Willis artery classes as a multi-class segmentation problem.

The original TopCoW label set was preserved in the submitted masks. Internally, the non-contiguous label 15 was mapped to a contiguous training label and then mapped back to label 15 before submission.

### Training Strategy

The model was trained from the visible labeled CTA training set using a single validation fold. Training used nnU-Net-style preprocessing, resampling, patch sampling, Dice-plus-cross-entropy optimization, and checkpointed long training. The selected checkpoint was a mature single-fold checkpoint at approximately 550 epochs.

### Inference Strategy

Inference was run on the 25 unlabeled CTA test volumes using the selected single-fold checkpoint. Predictions were generated as hard segmentation masks without test-time augmentation and without fold ensembling. Each prediction was restored to the original image geometry before submission.

### Post-processing

Post-processing was minimal. The main steps were geometry restoration, conversion from the internal contiguous label representation back to the TopCoW label set, and writing compressed NIfTI masks. No atlas fallback, connected-component filtering, or score-weighted ensemble was used in the final submission.

## Internal Validation

Internal validation used a held-out fold of 20 CTA cases. The selected single-fold nnU-Net checkpoint achieved an internal validation score of 0.6593, which was the strongest internal result available before finalization.

Submission completeness checks confirmed that all 25 test cases had readable, non-empty NIfTI masks, matched the expected submission rows, and contained valid labels: background plus labels 1 through 12 and 15.

## Official Test Result

The final official overall score was 10.33333, where lower is better.

Key official metrics:

- Dice: 0.62616
- clDice: 0.90425
- B0 error: 0.86866
- HD95: 16.45349
- F1 group 2: 0.52801
- Anterior graph accuracy: 0.40000
- Posterior graph accuracy: 0.40000
- Anterior topology: 0.40000
- Posterior topology: 0.40000
