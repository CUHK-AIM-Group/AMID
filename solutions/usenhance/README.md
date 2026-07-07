# USenhance 2023 Ultrasound Image Enhancement

## Challenge and Evaluation
USenhance 2023 is a paired ultrasound image enhancement task. The goal is to transform low-quality handheld ultrasound images into images closer to high-quality reference scans from clinical ultrasound systems. The submission contains one enhanced image for each test case, with evaluation based on LNCC, SSIM, and PSNR; the image-quality metrics are higher-better, while the leaderboard score reported here is mean position, where lower is better.

## Final Solution
### Method Overview
The final solution is a paired grayscale ultrasound restoration ensemble. It learns to map low-quality handheld ultrasound images to higher-quality reference-style images using paired supervision. Five independently trained U-Net generators are applied to every test image, and their outputs are averaged to reduce fold-specific texture noise. The submitted set contains 210 enhanced ultrasound images.

### Model Architecture
The generator is a pix2pix-style U-Net adapted to single-channel ultrasound. The encoder compresses the input image into multi-scale features, and the decoder reconstructs the enhanced image while skip connections preserve fine anatomical structure. Images are scaled to the range from -1 to 1, and the generator uses a tanh output layer. The final training recipe relies on paired reconstruction losses rather than an adversarial discriminator, prioritizing stable image fidelity over synthetic texture.

### Training Strategy
Training follows a 5-fold cross-validation protocol. For each fold, 672 paired images are used for training and 168 paired images are held out for validation, with no overlap. Each fold model is trained for 50 epochs with batch size 12. The loss combines strong pixel reconstruction with structural similarity: L1 loss has weight 100 and SSIM loss has weight 30. The adversarial loss weight is 0, so the final model behaves as a supervised restoration network rather than a GAN.

### Inference Strategy
At inference time, each of the five fold models predicts the enhanced image once normally and once with horizontal flip test-time augmentation. The flipped prediction is inverted back, averaged with the normal prediction, and then the five fold outputs are averaged uniformly. The final images are grayscale outputs at 256 by 256 resolution.

### Post-processing
Post-processing is intentionally minimal. The averaged image is clipped to the valid intensity range and converted back to standard grayscale format. No organ-specific rule, per-image metric tuning, or validation-based row selection is applied.

## Internal Validation
The final completeness check confirms 210 submitted rows and 210 enhanced images in the expected order. Internal fold records confirm five independently trained models, 672 training pairs per fold, 168 validation predictions per fold, and no overlap between fold training and validation sets. The recorded one-fold anchor score before the final five-fold ensemble was 0.3742965183.

## Official Test Result
- Leaderboard score / mean position: `11`
- Medal: none
- LNCC: `0.18658`
- SSIM: `0.38495`
- PSNR: `17.12416`
- LNCC position: `11`
- SSIM position: `11`
- PSNR position: `11`
- Percentile: `0`
