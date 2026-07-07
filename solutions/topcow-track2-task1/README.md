# TopCoW Track 2 Task 1: MRA Multi-class Segmentation

## Challenge and Evaluation

This challenge focuses on multi-class segmentation of the Circle of Willis and related cerebral arterial structures from MRA scans. The input is a 3D MRA volume, and the expected output is a voxel-wise segmentation mask assigning each vascular class to its corresponding anatomical label.

Evaluation is based on a lower-is-better overall rank-style score computed from multiple segmentation and topology-aware metrics. Key metrics include Dice overlap, centerline Dice, Betti-number topology error, Hausdorff distance, group-wise F1, and anterior/posterior graph or topology accuracy.

## Final Solution

### Method Overview

The final submission used a single 3D nnU-Net model trained for multi-class vessel segmentation. The selected model was initialized from a previously trained 50-epoch checkpoint and then continued to 100 total epochs. The final prediction was generated directly from this continued model on the public test set.

### Model Architecture

The model was a standard 3D full-resolution nnU-Net segmentation network. It used the self-configuring nnU-Net pipeline for preprocessing, patch-based 3D training, and multi-class softmax segmentation.

### Training Strategy

Training used the original 3D MRA training data with multi-class vessel labels. A 50-epoch nnU-Net checkpoint was used as initialization, and training was continued to 100 total epochs. This continuation strategy preserved the rare vessel classes better than a separate 100-epoch training run from scratch.

### Inference Strategy

Inference was performed on the 25 public test MRA volumes using the final 100-epoch continued checkpoint. Predictions were generated with nnU-Net sliding-window inference and test-time mirroring enabled.

### Post-processing

No additional topology correction, connected-component filtering, or ensemble post-processing was applied. The submitted masks were the direct nnU-Net multi-class segmentation outputs, exported in the required submission format.

## Internal Validation

Internal validation used a held-out set of 20 labeled MRA cases from the training data. The continued 100-epoch model achieved a mean Dice score of 0.7797 on this holdout set, improving over the corresponding 50-epoch model score of 0.7379.

Additional sanity checks confirmed that all 25 public test predictions were produced, image geometry matched the source MRA volumes, and the predicted labels covered the expected vessel classes from 0 through 12. Rare labels 8 and 9 were retained in the public test predictions.

## Official Test Result

The official public test submission was valid.

Final official overall score: 9.22222, where lower is better.

Key official metrics:

- Dice: 0.76087
- clDice: 0.90971
- Betti-0 error: 0.79858
- HD95: 10.9853
- Group-2 F1: 0.57883
- Anterior graph accuracy: 0.68
- Posterior graph accuracy: 0.36
- Anterior topology accuracy: 0.68
- Posterior topology accuracy: 0.36
