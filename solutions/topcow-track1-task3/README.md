# TopCoW Track 1 Task 3 - CTA Graph Edge Classification

## Challenge and Evaluation

This challenge evaluates Circle of Willis topology classification from 3D CTA brain angiography. For each test CTA volume, the required output is a JSON edge prediction describing whether predefined anterior and posterior Circle of Willis graph edges are present or absent.

The anterior graph contains four binary edges: L-A1, Acom, 3rd-A2, and R-A1. The posterior graph contains four binary edges: L-Pcom, L-P1, R-P1, and R-Pcom.

The official evaluation reports anterior and posterior topology accuracy, together with an overall rank-style score where lower values are better.

## Final Solution

### Method Overview

The final submission used a simple topology-prior method derived from the public training edge labels. Instead of training an image model, the method estimated the most frequent CTA graph topology in the labeled training set and applied that topology consistently to every public CTA test case.

The selected CTA topology was:

- Anterior pattern: L-A1 present, Acom present, 3rd-A2 absent, R-A1 present.
- Posterior pattern: L-Pcom absent, L-P1 present, R-P1 present, R-Pcom absent.

### Model Architecture

No trainable neural network was used for the final selected method. The solution is a deterministic frequency-based classifier over graph-edge variants.

The classifier has two independent categorical components:

- one anterior topology variant selected from observed anterior edge patterns;
- one posterior topology variant selected from observed posterior edge patterns.

Each component predicts the most common CTA variant observed in the public training labels.

### Training Strategy

Training consisted of counting binary anterior and posterior graph variants in the labeled CTA training cases. Each case contributed one anterior four-bit pattern and one posterior four-bit pattern.

The most frequent CTA anterior pattern was selected as the fixed anterior prediction, and the most frequent CTA posterior pattern was selected as the fixed posterior prediction. No image intensities, segmentations, or learned weights were used.

### Inference Strategy

At inference time, each CTA test case was assigned the same fixed topology prior. The method produced one prediction JSON per test case and a submission table linking each CTA case to its corresponding JSON file.

Because the official public test set contains CTA cases only, no cross-modality pairing was used at inference time.

### Post-processing

The post-processing step only enforced the required challenge output structure. Each prediction was written as a valid JSON object with anterior and posterior edge dictionaries using the official edge names.

A paired CT/MRA consistency rule was evaluated internally for paired validation data, but it was not applied to the final CTA-only test submission because no paired MRA test image was available.

## Internal Validation

Internal validation used a held-out fold with 40 validation cases and 160 training cases. On this paired internal validation setting, the topology-prior family with paired posterior consistency reached an internal score of 0.3697.

The internal component metrics were:

- anterior accuracy: 0.2500;
- posterior accuracy: 0.4895;
- overall internal accuracy: 0.3697.

For the final CTA-only export, the submission was checked for completeness and format validity: all 25 CTA test cases were present, every prediction file existed, and all anterior and posterior edge values were valid binary entries.

## Official Test Result

The official test submission was valid.

Official metrics:

- Anterior accuracy: 0.33333
- Posterior accuracy: 0.12500
- Overall official score: 5.5

The official score is rank-style with lower values better.
