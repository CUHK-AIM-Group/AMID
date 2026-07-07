# DENTEX Dental Enumeration and Diagnosis

## Challenge and Evaluation
DENTEX is a panoramic dental X-ray detection and diagnosis task. The submitted prediction for each case contains tooth bounding boxes together with quadrant, enumeration, and diagnosis labels. Evaluation uses AP/AR-style detection and classification metrics, where higher values are better; the leaderboard score reported here is mean position, where lower is better.

## Final Solution
### Method Overview
The final solution treats the task as tooth detection followed by anatomical normalization. A YOLO detector first finds proposed teeth and assigns preliminary diagnosis scores. A second calibration layer then organizes those detections into a plausible dental arch: boxes are mapped to quadrants and tooth numbers using learned position templates, and diagnosis labels are adjusted with statistics from the training set. This makes the output more consistent than using detector class scores alone, especially when neighboring teeth are close together or partially missing.

The submitted prediction set uses the most reliable calibrated detector output, chosen because it preserved a complete prediction structure for all test cases and avoided duplicated or malformed rows during final packaging.

### Model Architecture
The detector is a compact YOLOv8 model adapted to panoramic dental radiographs. It produces bounding boxes and diagnosis probabilities in one pass over the image. On top of the detector, a structured tooth-template module estimates where each FDI tooth position should lie in the panorama. The template layer uses median tooth locations learned from training images to convert unordered detector boxes into consistent quadrant and enumeration labels.

The final prediction is intentionally hybrid. High-confidence YOLO detections provide the main boxes, while low-confidence geometric priors add likely teeth that the detector may miss. Learned alternate tooth positions are used when two adjacent labels are commonly confused. Diagnosis fallbacks use majority statistics only when enough training examples support the replacement.

### Training Strategy
The detector weights come from a pretrained panoramic dental X-ray model, so the main training effort is calibration rather than detector training from scratch. Calibration statistics are fit from 564 training images. These statistics include median tooth locations, broad and compact geometry priors, diagnosis fallback tables, and common disagreement patterns between neighboring tooth labels.

The fallback rules are deliberately conservative. A majority diagnosis rule is used only when it has at least 30 supporting examples and the majority class accounts for at least 60% of them. Learned alternate tooth positions require at least 10 supporting examples and sufficient box overlap with the source tooth pattern. These thresholds keep the prior layers helpful without letting them overwhelm real detector evidence.

### Inference Strategy
Each panoramic image is processed independently. The detector is run with a large input size of 960 pixels and a low confidence threshold of 0.01 to favor recall, while non-maximum suppression uses IoU 0.5 and keeps at most 80 detections. Test-time augmentation is enabled so that weak teeth have multiple chances to be detected.

After detection, the calibration layer assigns FDI quadrant and enumeration labels from the learned dental-arch templates. It then adds very low-score prior boxes for anatomically likely teeth, with broader priors receiving lower scores than compact priors. Bounded alternate labels are inserted only when the source detection and learned tooth-position pattern satisfy overlap and score constraints.

### Post-processing
Post-processing balances recall with ranking quality. A compact-prior layer that was too dominant is removed before final alternate composition. Prompt-guided alternates and same-quadrant tooth alternates are retained at very low confidence so they can help recall without displacing strong detector boxes. Predictions above 0.01 confidence are capped to the top 40 per image, while low-score anatomical alternates are preserved. The final prediction set contains 14,288 tooth predictions across 141 cases.

## Internal Validation
The final completeness check recorded 141 submitted cases, all matching the expected sample order. No prediction file was missing, no invalid field was found, and the final count was 14,288 tooth predictions. The selected calibrated detector output was kept because it produced a valid, complete case set.

## Official Test Result
- Leaderboard score / mean position: `3.5`
- Medal: bronze
- AP mean: `0.48825`
- AP50 mean: `0.64702`
- AP75 mean: `0.58692`
- AR mean: `0.68248`
- AP quadrant: `0.59302`
- AP enumeration: `0.28721`
- AP diagnosis: `0.58453`
- Percentile: `0.75`
