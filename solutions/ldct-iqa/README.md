# Low-Dose CT Perceptual Image Quality Assessment

## Challenge and Evaluation
This task is no-reference perceptual image quality assessment for low-dose CT images. The submission assigns one scalar quality score to each image. Official evaluation uses PLCC, SROCC, KROCC, and an overall correlation-based metric, where higher correlation is better; the leaderboard score reported here is mean position, where lower is better.

## Final Solution
### Method Overview
The final solution predicts image quality as a scalar regression problem. It uses a five-fold ensemble of compact vision-transformer regressors. Each fold combines two complementary views of the CT image: one model focuses on the diagnostically important central crop, while the other sees the full resized image. Their predictions are blended within each fold and then averaged across folds for the final score.

### Model Architecture
The regressor uses a convolutional stem followed by a lightweight vision-transformer body and a scalar output head. The convolutional stem captures local CT texture, while the transformer layers aggregate broader spatial quality cues. Two members share this general design but differ in input view. The crop-focused member emphasizes central anatomy and noise texture after cropping; the resize member preserves global image context. Both output a single perceived-quality score.

### Training Strategy
Each fold trains the scalar regressors for 7 epochs using AdamW with learning rate 5e-5. The loss combines L1 and mean-squared error on normalized quality scores, encouraging both robust ranking and accurate score magnitude. The final model uses fixed fold weights and fixed crop/full-image member weights; no image-specific tuning is performed after training.

### Inference Strategy
Inference uses seven-view test-time augmentation for each image, averaging predictions across the augmented views to reduce sensitivity to small spatial changes. Within each fold, the crop-focused member receives 60% weight and the full-image member receives 40% weight. The five fold predictions are then uniformly averaged to obtain one final quality score per CT image.

### Post-processing
The scalar predictions are clipped to the valid 0 to 4 range and rounded to a 0.2-point grid, matching the granularity of the target radiologist-quality scale. There is no spatial post-processing because the task output is a single number per image.

## Internal Validation
Each fold contains 200 validation predictions and 300 test predictions. The arithmetic mean of the internal fold metrics is 2.8397675635176114, and the final test output matches the planned uniform fold average.

## Official Test Result
- Leaderboard score / mean position: `2`
- Medal: silver
- PLCC: `0.95598`
- SROCC: `0.95354`
- KROCC: `0.82799`
- Overall metric: `2.73751`
- Percentile: `0.83333`
