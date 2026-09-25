# Eye Disease Classification — Leakage-aware Kaggle Experiment

Dataset: 11,839 fundus images, 8 classes (ODIR5K, APTOS2019, ACRIMA, ORIGA). Run this notebook in Kaggle with the attached dataset and GPU.

## Method
- Group-aware split (seed 42): train 9468, validation 1187, test 1184.
- ODIR left/right images for the same patient are always assigned to the same split. ACRIMA Im identifiers are conservatively grouped. APTOS2019 and ORIGA lack verified patient identifiers.
- Class weighting calculated only on the corrected training split. Scratch CNN trained from scratch; ImageNet ResNet50 head training then fine-tuning of late backbone layers. Same corrected splits and class mapping for both.
- Best checkpoints chosen by validation loss. Test set used only for final evaluation after both models are fixed.

## Results (corrected split; do not mix with earlier image-level results)
| Model               |   Test Accuracy |   Macro F1 |   Weighted F1 |   Batch Size |   Mean Batch Latency (ms) |   Median Batch Latency (ms) |   Throughput (images/sec) |   Parameters |   Checkpoint Size (MB) |
|:--------------------|----------------:|-----------:|--------------:|-------------:|--------------------------:|----------------------------:|--------------------------:|-------------:|-----------------------:|
| Scratch CNN         |          0.4502 |     0.3883 |        0.5330 |           32 |                   35.4336 |                     35.3342 |                  903.0980 |       458184 |                 5.3313 |
| Fine-tuned ResNet50 |          0.5068 |     0.4268 |        0.5686 |           32 |                  211.7701 |                    210.6820 |                  151.1073 |     24114312 |               206.7428 |

Per-class performance: `results/scratch_cnn/classification_report.csv`, `results/resnet50/classification_report.csv`, and `results/per_class_comparison.csv`. Confusion matrices and training curves are in the corresponding model directories under `results/`. Single-image latency: `results/single_image_latency.csv`.

## Deployment interpretation
Benchmark latency is measured on this Kaggle session's GPU. Batch-32 inference excludes disk reads and external preprocessing; the separate batch-1 benchmark is also provided. Checkpoint size includes Keras serialization and possible optimizer state, not only deployed weight size. Compare accuracy, per-class F1/recall, latency and footprint together; neither model is clinically validated.

## Known limitations
- ODIR group overlap was found in the original provided split, so original test scores should not be presented as independent final evaluation.
- Verified patient identifiers are unavailable for APTOS2019 and ORIGA; absence of all patient-level leakage cannot be guaranteed.
- Minority categories, especially Hypertension, have very small sample sizes. Class weights may improve recall at the cost of precision.
- Publicly shared medical images must have appropriate redistribution rights and de-identification.

## Reproduce
Attach Kaggle dataset `nebalelshobary/eye-disease-classification-dataset`, enable GPU and (if needed) Internet for ImageNet weights; run cells top-to-bottom. Results are written under `/kaggle/working/outputs/leakage_aware/`. Save a Kaggle version to persist checkpoints and download output assets for GitHub. Do not commit the 3.8 GB image dataset or secret tokens to GitHub.
