# TopCoW Track 2 Task 3 MRA Graph Edge Classification

## Challenge and Evaluation
TopCoW Track 2 Task 3 is a Circle of Willis topology classification task on 3D MRA volumes. For each test case, the submission predicts eight binary graph edges: four anterior edges (`L-A1`, `Acom`, `3rd-A2`, `R-A1`) and four posterior edges (`L-Pcom`, `L-P1`, `R-P1`, `R-Pcom`). Evaluation reports anterior accuracy and posterior accuracy, then converts them into leaderboard positions; the final score is the mean position, where lower is better.

## Final Solution
### Method Overview
The final solution is a segmentation-to-topology pipeline. Instead of training a direct graph classifier on the full MRA volume, it first predicts an anatomical Circle of Willis vessel segmentation and then converts the vessel labels into edge presence or absence. Five independently trained 3D SegResNet models are applied to every test case. Their anterior-edge predictions are combined by majority voting, while posterior edges are constrained by a training-set anatomical prior because the learned posterior predictions were less reliable in internal validation.

The submitted set contains 25 MRA cases. The final anterior predictions are concentrated in three patterns: 12 cases have all four anterior edges present; 9 cases have left A1, Acom, and right A1 present but no third-A2 edge; and 4 cases have left A1, third-A2, and right A1 present but no Acom edge. All 25 cases use the same posterior prior: absent left Pcom, present left P1, present right P1, and absent right Pcom.

### Model Architecture
The image model is a compact 3D MONAI SegResNet used as an ROI segmentation backbone. It takes a single-channel MRA crop and predicts a multi-class vessel mask with the following anatomical classes: background, basilar artery, left and right posterior cerebral arteries, left and right internal carotid arteries, left and right middle cerebral arteries, left and right posterior communicating arteries, anterior communicating artery, left and right anterior cerebral arteries, and third A2.

The selected high-resolution model starts with 12 convolutional filters, uses a shallow residual encoder-decoder layout, applies 0.1 dropout, and uses group normalization. It has one input channel and one output channel per vessel label. There is no separate graph-neural-network head; topology is derived from the predicted vessel mask using label presence and local 3D contact tests between anatomically connected labels.

### Training Strategy
Training follows a 5-fold cross-validation protocol. Each fold trains one independent SegResNet model on 160 labeled cases while excluding the 40 cases in that fold's validation bucket. The selected model family uses 80 epochs per fold, for 400 total completed epochs, with batch size 1, learning rate 2e-4, AdamW optimization with weight decay 1e-5, cosine annealing learning-rate scheduling, and mixed-precision training.

The ROI crop is padded by 16 voxels around the Circle of Willis region and resized to 128 by 128 by 64 voxels. Intensities are clipped using nonzero-voxel percentiles and normalized by nonzero mean and standard deviation. Training uses Dice plus cross-entropy loss with one-hot targets and softmax probabilities. Class weights are inverse-square-root frequency weights, clipped between 0.05 and 20.0, with background fixed to 0.05; this increases pressure on small-vessel labels such as Pcom, Acom, and third A2 while keeping the background from dominating the loss.

Data augmentation is intentionally light: random flips over two spatial axes and small intensity jitter are used during training. The trained model for each fold is selected by training loss within that fold. No earlier-stage model is reused, and validation labels or validation ROI boxes are not used for held-out prediction generation.

### Inference Strategy
At inference time, each MRA volume is cropped to the Circle of Willis region when available; otherwise a median training-region estimate is used as a fallback. The crop is resized to 128 by 128 by 64 voxels, normalized with the same nonzero-voxel rule as training, and passed through the SegResNet. The predicted vessel label is the highest-probability class at each voxel. The predicted crop mask is resized back to the original crop resolution before topology extraction.

For each of the five trained models, the predicted mask is converted into eight binary graph edges. The anterior edges combine label presence with explicit contact checks: `L-A1` requires left ACA and left ICA contact, `R-A1` requires right ACA and right ICA contact, `Acom` is present when the Acom label is present, and `3rd-A2` is present when the third-A2 label is present. The posterior edges use analogous anatomical evidence: Pcom edges are read from the left and right Pcom labels, while `L-P1` and `R-P1` require PCA-to-basilar contact.

The final test prediction is a fold ensemble over the five edge outputs. The anterior result is majority-voted edge by edge across the five models. The posterior result is fixed to the visible-training majority prior because validation showed that this prior was more stable than the learned posterior model output.

### Post-processing
Post-processing is deterministic and topology-aware. Degenerate masks with too few positive edges fall back to the global topology prior. The anterior topology uses edge-wise fold consensus. The posterior topology is projected to the prior pattern of absent left Pcom, present left P1, present right P1, and absent right Pcom. The output is then written as one edge dictionary per case with separate anterior and posterior fields.

## Internal Validation
The accepted internal validation run uses the 5-fold protocol described above. The recorded fold scores are 0.2797619048, 0.375, 0.7, 0.2371794872, and 0.1571428571, giving a fold mean of 0.3498168498 with fold standard deviation 0.1886693234. The fold checks record 160 training cases and 40 validation cases per fold, no held-out IDs in training, five independently trained models, no earlier-stage model reuse, and no validation-region use during held-out prediction.

The final completeness check recorded 25 submitted rows and 25 prediction files, matching the expected public-test case count.

## Official Test Result
- Leaderboard score / mean position: `5`
- Medal: none
- Anterior accuracy: `0.46296`
- Posterior accuracy: `0.16667`
- Anterior accuracy position: `3`
- Posterior accuracy position: `7`
- Percentile: `0.33333`
