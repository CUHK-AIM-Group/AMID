# TopCoW Track 1 Task 2: CTA 3D Bounding Box Detection

## Challenge and Evaluation

This challenge asks participants to localize the Circle of Willis region in three-dimensional computed tomography angiography volumes. The input is a CTA volume, and the required output is one 3D bounding box per case, represented by its voxel-space size and location. The final submission consists of a table linking each CTA case to a JSON file containing the predicted bounding box.

The task is evaluated by spatial overlap between predicted and reference boxes. The official report includes 3D intersection-over-union and boundary IoU. Higher IoU and boundary IoU indicate better localization, while the official overall score is rank-derived and lower is better.

## Final Solution

### Method Overview

The final method is a five-fold ensemble of lightweight 3D heatmap center predictors. For each fold, two center-prediction models are trained with the same crop and size prior but different coordinate-loss weighting. Their fold-level predictions are averaged, and the final test prediction is the rounded uniform average of the five fold-level bounding boxes.

The method separates box center estimation from box size estimation. The neural networks predict the center offset of the Circle of Willis region within a coarse crop. The box dimensions are supplied by a training-set geometry prior, which stabilizes the predicted box size and reduces sensitivity to noisy heatmap peaks.

### Model Architecture

Each model is a compact 3D convolutional heatmap network. The network receives a normalized CTA crop together with three coordinate channels. It uses several 3D convolution blocks with group normalization and SiLU activations, followed by a one-channel heatmap head. A soft-argmax over the heatmap gives a continuous center offset. The base width is 16 channels, and inference operates on downsampled crops of size 48 x 48 x 24.

### Training Strategy

Training uses 100 labeled CTA cases split into five folds of 20 cases each. For each fold, two models are trained on the remaining 80 cases and validated on the held-out fold. Both models use cropped CTA volumes centered from a training-geometry prior, percentile intensity clipping between the 1st and 99th percentiles, and min-max normalization to [0, 1].

The models are trained for 160 epochs with batch size 4, AdamW optimization, learning rate 0.001, and weight decay 0.0001. The heatmap target uses a Gaussian center objective with sigma 0.04. A coordinate loss is added with weight 20.0. The first model uses equal coordinate-axis weights, while the second gives extra emphasis to the z axis.

### Inference Strategy

For each test CTA volume, each fold-specific model predicts a center offset inside a geometry-prior crop. This offset is converted back to a full-volume center location. The predicted box size comes from the fold training set's geometry prior rather than from the heatmap model itself.

Within each fold, the two model predictions are averaged to obtain one fold-level bounding box per test case. The final prediction then averages the five fold-level boxes. Sizes and locations are rounded to integer voxel coordinates.

### Post-processing

Post-processing is intentionally simple. Predicted locations are clipped so that the box remains inside the image volume. Final box sizes and locations are produced by rounded uniform averaging across folds. No case-specific threshold tuning or manual correction is used.

## Internal Validation

The selected solution was validated with a locked five-fold split over the 100 labeled CTA cases. Each fold held out 20 cases, and every final fold model was trained without its held-out cases. The selected five-fold candidate reached an internal validation score of 0.68439 under the bounding-box evaluation used for model selection.

The final test export was checked for completeness before submission. It contained predictions for all 25 test CTA cases, with one bounding-box JSON file per case and a valid submission table referencing every prediction.

## Official Test Result

The final official overall score was 4.5, where lower is better for the rank-derived score. The official boundary IoU was 0.66907, and the official 3D IoU was 0.70745. The boundary-IoU position was 2nd, the IoU position was 7th, and the mean position was 4.5. The submission was valid and ranked above the median.
