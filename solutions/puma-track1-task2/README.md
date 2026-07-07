# PUMA Track 1 Task 2 Nuclei Detection, 3 Classes

## Challenge and Evaluation
PUMA Track 1 Task 2 is three-class nuclei detection in H&E melanoma regions of interest. The classes are tumor, TILs, and other. Evaluation uses 15-pixel centroid matching, then computes per-class F1 and macro F1; higher F1 is better.

## Final Solution
### Method Overview
The final solution separates nuclei localization from nuclei classification. CellViT first proposes nuclei centroids from the H&E image. A second classifier then assigns each proposed nucleus to tumor, TILs, or other using local token features and neighborhood context. Five fold-level detection sets are merged into one final centroid list.

### Model Architecture
The detection stage uses CellViT-256 features from high-magnification histology patches. CellViT provides proposed nuclei and token-level visual embeddings. The classification stage is a histogram-gradient-boosting model that reads token features around each nucleus, 64-pixel local context, and a broader 128-pixel neighborhood summary of raw neighboring nucleus types. A fixed probability bias toward the "other" class is applied before filtering to reduce over-calling tumor or TILs when the local evidence is ambiguous.

### Training Strategy
The token classifier is trained with up to 80 training cases and a balanced cap of up to 9,000 examples per class. The boosting model uses 200 iterations, learning rate 0.05, 31 maximum leaf nodes, and no additional class weighting. Feature extraction is accelerated with mixed precision, while the classifier itself remains deterministic.

### Inference Strategy
Each fold produces a list of nuclei centroids, class probabilities, and confidence values. The same feature representation, class bias, and confidence threshold are used for every fold. During ensembling, nearby detections within a 6-pixel radius are clustered together. Class probabilities and confidence scores from all contributing fold detections are averaged to assign the final class.

### Post-processing
The final confidence filter keeps nuclei with confidence at least 0.35. This threshold is applied consistently across folds before merging, so the final output is a uniform five-fold ensemble rather than a hand-picked set of detections.

## Internal Validation
Internal records show that one representative training fold used 80 selected cases with 8,698 TIL samples, 6,098 other-cell samples, and 9,000 tumor samples. The final prediction set contains 41 cases and 18,663 nuclei. A standalone numeric cross-validation macro-F1 summary was not preserved for this final submission.

## Official Test Result
- Leaderboard score / mean position: `11`
- Medal: none
- Macro F1: `0.54359`
- F1 TILs: `0.51768`
- F1 other: `0.36825`
- F1 tumor: `0.74484`
