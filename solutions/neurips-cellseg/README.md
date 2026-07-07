# NeurIPS 2022 Cell Segmentation Challenge

## Challenge and Evaluation
This challenge evaluates cell instance segmentation in microscopy images. The submission contains one instance-labeled mask per image. Official scoring reports F1 at several IoU thresholds, where higher F1 indicates better instance localization, separation, and boundary quality; the leaderboard score reported here is mean position, where lower is better.

## Final Solution
### Method Overview
The final solution fine-tunes Cellpose-SAM for microscopy cell instance segmentation. Cellpose-SAM already provides a strong general-purpose cell segmentation prior; the challenge training masks are used to adapt it to the specific cell appearances and annotation style of this dataset. The same fine-tuned model and the same inference thresholds are then applied consistently to all validation and test images.

### Model Architecture
The model is the Cellpose-SAM instance segmentation architecture. It predicts individual cell masks directly, rather than producing a semantic foreground map that would need a separate watershed step. Input normalization is enabled. RGB microscopy images are processed as three-channel images, while grayscale images are processed as single-channel images.

### Training Strategy
The model is fine-tuned for 100 epochs with batch size 1, learning rate 1e-5, and weight decay 0.1. Model weights are saved periodically during training. The final selected recipe uses the fine-tuned Cellpose-SAM weights with one fixed inference policy for every image: no fold-specific score weighting, no per-fold row selection, and no validation-image-specific adjustment.

### Inference Strategy
Inference is tiled so that large microscopy images can be processed at stable memory cost. The tile size is 256 pixels with 20% tile overlap, batch size 8, automatic diameter estimation, flow threshold 0.4, and cell-probability threshold 0. Instances smaller than 45 pixels are removed. Test-time augmentation is disabled to keep the predictions deterministic. A safety cap of 10,000 instances per mask prevents invalid oversized outputs.

The final five-fold output uses the same fine-tuned model and calibration settings across folds, so the ensemble acts as a consistency wrapper around one selected Cellpose-SAM recipe rather than a set of independently tuned variants.

### Post-processing
Post-processing is limited to instance cleanup and output formatting. Very small masks are removed, labels are limited to the largest 10,000 instances when necessary, and the final masks are written as lossless TIFF files. Sixteen-bit integer masks are used when possible, with a larger integer type only when the number of instances requires it. The final test inference produced 221 prediction masks.

## Internal Validation
The internal accepted score for this Cellpose-SAM solution is 0.893655. Internal checks also confirm that the same trained model and inference settings are used consistently across the five validation folds and the final test prediction set.

## Official Test Result
- Leaderboard score / mean position: `1.6`
- Medal: silver
- F1 @ IoU 0.5: `0.89501`
- F1 @ IoU 0.6: `0.86342`
- F1 @ IoU 0.7: `0.8035`
- F1 @ IoU 0.8: `0.68415`
- F1 @ IoU 0.9: `0.41917`
- Percentile: `0.94`
