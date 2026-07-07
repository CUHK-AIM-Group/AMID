# PUMA Track 1 Task 1 Semantic Tissue Segmentation

## Challenge and Evaluation
PUMA Track 1 Task 1 is semantic tissue segmentation for H&E melanoma regions of interest. Each submission contains one raster tissue mask per case. The mask predicts background plus five tissue categories: stroma, blood vessel, tumor, epidermis, and necrosis.

The official tissue-segmentation metric is micro Dice, where higher is better. The leaderboard score reported by the benchmark is mean position, where lower is better. The final result below therefore reports both the official Dice value and the corresponding mean-position leaderboard score.

## Final Solution
### Method Overview
The final solution is a locked five-fold ensemble based on frozen UNI pathology foundation features. Instead of training a full dense segmentation network end to end, each image is converted into a pyramid of UNI patch-token features, and lightweight multi-layer perceptron heads learn to classify local tissue regions from those frozen features. This keeps the histology representation stable while allowing the task-specific classifier to adapt to the PUMA tissue labels.

The selected final submission uses three complementary patch heads. The strongest member uses RGB-and-coordinate auxiliary features and stain-dark feature augmentation during training. Two companion members use RGB auxiliary features without stain augmentation. Their outputs are combined at inference time, giving a compact ensemble that benefits from different random initializations and feature augmentations without changing the underlying UNI encoder.

### Model Architecture
The backbone is a frozen UNI ViT-L/16 encoder. Each 1024 x 1024 region of interest is analyzed through a pyramid crop strategy rather than a single global view. The pyramid representation lets the classifier see both broad tissue context and more localized visual evidence.

For every crop view, UNI produces a 14 x 14 patch-token grid. A lightweight MLP head maps each patch token, optionally concatenated with auxiliary color and spatial-coordinate features, into six-class tissue logits. The six output labels correspond to background, stroma, blood vessel, tumor, epidermis, and necrosis. The head is deliberately small compared with the frozen foundation encoder, so the trainable part of the model focuses on learning the label readout rather than relearning histology features.

### Training Strategy
Training follows the audited five-fold protocol. For each fold, the validation cases are exactly the locked fold bucket, and the training cases are all remaining labeled cases. This produces one fold-specific head per ensemble member while keeping the same architecture, features, and inference recipe across all folds.

Each MLP head is trained for 60 epochs with batch size 4096 over patch tokens. The loss uses soft patch-label distributions rather than only hard majority labels, preserving information from mixed tissue patches at class boundaries. Class weighting is controlled with a power of 0.65 to reduce domination by frequent tissue/background regions while avoiding excessive amplification of rare labels. The primary head additionally uses stain-dark feature augmentation during training, which improves robustness to H&E staining variation. The two companion heads provide diversity through independent training seeds and RGB-only auxiliary features.

During stage 2, the promoted method variant was audited under the locked five-fold protocol. The final accepted method variant achieved an internal five-fold mean Dice of `0.57221`, with fold Dice scores of `0.57058`, `0.62009`, `0.49498`, `0.53696`, and `0.63846`.

### Inference Strategy
At inference time, each fold loads the three corresponding fold-specific heads. For every image, the same pyramid crop feature extraction is applied, and the three heads produce tissue logits over the patch grid. A small logit-prior adjustment with alpha `0.05` is applied before ensembling to stabilize class frequency behavior. The three heads are then averaged in logit space.

The final submission also uses flip test-time augmentation. Predictions are generated for the original view and horizontal/vertical flip variants, the augmented predictions are restored to the original orientation, converted to probabilities, and averaged. The probability-averaged output is then reduced to the final class map by argmax.

### Post-processing
The only morphology step in the final accepted export is strict isolated-hole filling. After the patch-grid prediction is merged and resized to the required 1024 x 1024 mask, single-cell holes whose eight neighbors agree on the same non-background class are filled. Broader near-hole filling and fold-specific repair rules are not used in the final submission. This keeps the post-processing conservative: it removes isolated grid errors while avoiding class-specific or case-specific overcorrection.

## Internal Validation
The final submission passed format validation with 41 submitted cases and one mask per case. The accepted final candidate is anchored to the best audited stage-2 method variant, whose locked five-fold mean Dice was `0.57221`. The stage-2 review confirmed that all five folds were present, each fold predicted only its locked validation bucket, each fold trained on the complement of that bucket, and the same fold-invariant ensemble/TTA/post-processing recipe was used throughout.

## Official Test Result
- Leaderboard score / mean position: `9.0`
- Medal: none
- Dice: `0.56395`
- Percentile: `0.2`
