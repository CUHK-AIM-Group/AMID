# PUMA Track 2 Task 1 Semantic Tissue Segmentation

## Challenge and Evaluation
PUMA Track 2 Task 1 is semantic tissue segmentation for H&E melanoma regions of interest. The submission contains one multi-class tissue mask per case. The primary official metric is micro Dice, where higher is better.

## Final Solution
### Method Overview
The final solution is a five-fold six-class tissue segmentation ensemble built on Phikon-v2 histopathology features. Unlike the Track 1 tissue model, this version fine-tunes the final part of the foundation encoder as well as the segmentation head. The prediction combines global image context with local crop evidence, then smooths the class map before fold ensembling.

### Model Architecture
The backbone is Phikon-v2, a vision foundation model for pathology images. The final backbone block is fine-tuned while earlier layers remain fixed, giving the model some task adaptation without fully retraining the encoder. A linear token head predicts six tissue-class logits on a coarse grid. The model is evaluated in three complementary ways: on the full image, on fixed local crops, and on overlapping local crops.

### Training Strategy
Training runs for 18 epochs with batch size 4. The segmentation head uses learning rate 0.002, while the partially fine-tuned backbone uses the much smaller learning rate 1e-5. Class weights are derived from inverse token frequency so that rare tissue types influence the loss more strongly; the background weight is reduced to avoid dominating training.

### Inference Strategy
Inference uses dihedral test-time augmentation: horizontal and vertical flips, 90/180/270-degree rotations, and diagonal flips. Logits from all augmented views are inverted back to the original orientation and averaged. Coarse token logits are upsampled with bicubic interpolation. The final fold prediction blends full-image evidence equally with the average of fixed local crops and overlapping 640-pixel local crops, so both global tissue layout and local texture affect the result.

### Post-processing
The predicted class map is smoothed with small majority filters: first a 3-by-3 filter, then a 5-by-5 filter. This removes isolated single-pixel class noise while preserving tissue regions. The five fold masks are converted to one-hot maps, averaged uniformly, and converted back to labels, which is equivalent to majority voting with a deterministic tie break.

## Internal Validation
Internal records confirm that five fold models were trained and used. One representative fold records 132 training cases and 32 validation predictions, together with the class-weighting statistics. A standalone numeric cross-validation Dice summary was not preserved for this final submission.

## Official Test Result
- Leaderboard score / mean position: `9`
- Medal: none
- Dice: `0.56214`
- Percentile: `0.2`
