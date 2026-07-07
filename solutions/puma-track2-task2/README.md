# PUMA Track 2 Task 2 Nuclei Detection, 10 Classes

## Challenge and Evaluation
PUMA Track 2 Task 2 is ten-class nuclei detection in H&E melanoma regions of interest. The target classes include tumor, lymphocytes, epithelium, histiocytes, melanophages, apoptotic cells, neutrophils, plasma cells, stromal cells, and endothelium. Evaluation uses centroid matching followed by macro F1 and per-class F1; higher F1 is better.

## Final Solution
### Method Overview
The final solution is a two-stage nuclei detection and classification pipeline. First, PathoSAM proposes individual nuclei instances from the H&E image. Second, each nucleus is classified into one of the ten target classes using crop-level pathology embeddings from the Midnight foundation model. The final submitted set uses the most complete and reliable fold prediction set from this pipeline.

### Model Architecture
The instance proposal stage uses the histopathology-adapted PathoSAM model. Images are processed as RGB regions at native 1024 by 1024 resolution using tiled inference with overlap, so nuclei near tile borders still receive complete context. PathoSAM outputs proposed nucleus masks and centroids.

For classification, a crop centered on each nucleus is embedded by the frozen Midnight pathology encoder. The embedding is passed to a small ensemble of classical classifiers: random forest, extremely randomized trees, and a logistic component. This combination provides nonlinear decision boundaries while keeping class probabilities stable.

### Training Strategy
For each fold, the classifier is trained from 6,000 sampled nuclei while excluding that fold's validation cases. The PathoSAM proposal model is used as the instance generator, and Midnight embeddings provide the classification features. No extra per-class calibration layer is added after the classifier ensemble; the final labels come from the learned crop classifier probabilities.

The final submitted prediction set uses the fold output that completed cleanly for all 41 test cases and produced the best internal support among the available nuclei JSON sets.

### Inference Strategy
For each test image, PathoSAM proposes nuclei and removes implausibly small or large objects. Accepted nuclei must have area between 20 and 1600 pixels. Nearby duplicate proposals are suppressed with a 12.5-pixel non-maximum-suppression radius, and each image is capped at 1,250 nuclei. The final selected predictions contain between 224 and 822 nuclei per case, with a median of 437.

### Post-processing
Post-processing removes nuclei whose centroids are within 2 pixels of the image boundary, because these are often incomplete tile-edge detections. Confidence values are slightly scaled for numerical stability. Coordinates, labels, and confidence fields are otherwise preserved from the PathoSAM-Midnight prediction pipeline.

## Internal Validation
Internal validation records a best one-fold F1 of 0.4908803642 and a five-fold internal F1 of 0.4211389793 for this PathoSAM-Midnight approach. These scores support using the selected complete fold prediction set for final submission.

## Official Test Result
- Leaderboard score / mean position: `1`
- Medal: gold
- Macro F1: `0.28422`
- F1 epithelium: `0.41428`
- F1 histiocytes: `0.23294`
- F1 melanophages: `0.22822`
- F1 apoptotic cells: `0.14558`
- F1 neutrophils: `0.087`
- F1 lymphocytes: `0.45925`
- F1 tumor: `0.81275`
- F1 plasma cells: `0.09488`
- F1 stromal cells: `0.14795`
- F1 endothelium: `0.21931`
