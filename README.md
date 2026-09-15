# PlantVillage Multi-Task Vision Research

A reproducible computer-vision study on the PlantVillage dataset covering:

- **Classification**
- **Segmentation**
- **Leaf-level disease detection**

The project is designed as a unified research workflow rather than three unrelated notebooks. Outputs from the segmentation study are reused to support the detection and classification components.

---

## Project Status

| Component | Status |
|---|---|
| Dataset auditing / pairing | ✅ Complete |
| Segmentation preprocessing / QC | ✅ Complete |
| Segmentation experiments | ✅ Complete |
| Detection preprocessing / QC | ✅ Complete |
| Detection experiments | ✅ Complete |
| Classification preprocessing / QC | ✅ Complete |
| Classification experiments | ✅ Complete |
| Cross-model comparisons | ✅ Complete |
| Final conference-paper writing | 🔄 Next |

---

## Dataset Lineage

Initial PlantVillage audit:

```text
Color images      : 54,305
Grayscale images  : 54,305
Segmented images  : 54,306
Classes           : 38
```

One unmatched segmented sample was identified. A two-stage filename-matching process produced 54,305 unique color–segmented pairs.

One unusable source pair was subsequently excluded, producing a 54,304-sample source set for the final classification experiments.

Different tasks use different final sample counts because their supervision requirements differ.

| Task | Final Samples | Train | Validation | Test |
|---|---:|---:|---:|---:|
| Classification | 54,304 | 43,443 | 5,430 | 5,431 |
| Segmentation | 52,969 | 42,371 | 5,300 | 5,298 |
| Detection | 52,959 | 42,363 | 5,300 | 5,296 |

---

# 1. Segmentation

## Goal

Binary leaf segmentation using PlantVillage color images as input and QC-approved dataset-provided segmented images as the supervision source.

## Preprocessing Highlights

- Two-stage image pairing
- Fixed stratified split
- Leakage checks
- spatial-dimension QC
- one unusable source exclusion
- segmented-source background QC
- dynamic binary target generation
- threshold = 15
- nearest-neighbor mask alignment when required

Final segmentation set:

```text
52,969 samples
38 classes
42,371 train
5,300 validation
5,298 test
```

## Models

```text
U-Net + ResNet34
DeepLabV3+ + ResNet34
FPN + ResNet34
```

## Final Test Results

| Model | Dice | IoU |
|---|---:|---:|
| **U-Net** | **0.991319** | **0.982788** |
| FPN | 0.990844 | 0.981854 |
| DeepLabV3+ | 0.990106 | 0.980405 |

**Best observed segmentation model:** U-Net.

The differences are small; the repository reports an observed experimental ranking rather than universal architectural superiority.

---

# 2. Leaf-Level Disease Detection

## Goal

Whole-leaf localization with PlantVillage disease-class prediction.

This is **not lesion-level detection**.

## Why New Detection Labels Were Needed

PlantVillage does not provide suitable native bounding-box annotations for this workflow.

Early full-image pseudo-box experiments were rejected from the final methodology because every object occupied the complete image and did not provide meaningful localization supervision.

The final boxes were generated from QC-approved leaf masks.

## Detection Dataset

```text
52,959 samples
38 classes
42,363 train
5,300 validation
5,296 test
```

Fifteen suspicious mask-derived boxes were manually reviewed:

```text
5 retained
10 excluded
```

## Transfer-Learning Pipeline

All final detectors use:

```text
COCO pretrained
      ↓
PlantDoc object detection
      ↓
Best PlantDoc checkpoint
      ↓
PlantVillage tight leaf boxes
```

## Models

```text
YOLOv8n
YOLOv8s
YOLO11n
```

## Final Test Results

| Model | Precision | Recall | F1 | mAP50 | mAP50-95 |
|---|---:|---:|---:|---:|---:|
| YOLOv8n | 0.992832 | 0.991846 | 0.992339 | 0.994014 | 0.981625 |
| **YOLOv8s** | **0.994737** | **0.995709** | **0.995223** | **0.994231** | **0.983002** |
| YOLO11n | 0.993128 | 0.991207 | 0.992167 | 0.993787 | 0.978832 |

**Best observed detection model by mAP50-95:** YOLOv8s.

---

# 3. Classification

## Motivation

Ordinary PlantVillage classification is highly saturated. The final classification study therefore focuses on **robustness to background intervention**, not only on normal RGB accuracy.

## Classification Source

```text
54,304 samples
38 classes
43,443 train
5,430 validation
5,431 test
```

The stricter segmentation subset was not used because it caused one class to have zero test samples.

The completed U-Net segmentation model was used to predict a leaf mask for all 54,304 classification images.

## Four Views

```text
Original RGB
Leaf-only
Background-only
Counterfactual background
```

## Experiments

```text
Standard RGB EfficientNet-B0
Leaf-Only Teacher EfficientNet-B0
MG-CCD EfficientNet-B0
```

### Proposed MG-CCD Strategy

MG-CCD = **Mask-Guided Counterfactual Consistency Distillation**

Training combines:

```text
Original RGB cross-entropy
+ Counterfactual cross-entropy
+ Leaf-only teacher distillation
+ Original/Counterfactual consistency
```

The final MG-CCD student requires **RGB input only** at inference time.

## Final Multi-View Macro-F1

| Model | Original | Leaf-only | Counterfactual | Background-only |
|---|---:|---:|---:|---:|
| Standard RGB | **0.997589** | 0.648865 | 0.868435 | 0.040040 |
| Leaf-Only Teacher | 0.950641 | **0.996449** | 0.957307 | 0.080980 |
| MG-CCD | 0.996907 | 0.986897 | **0.995904** | 0.088776 |

## Main Classification Finding

Standard RGB:

```text
Original → Counterfactual Macro-F1 drop
= 12.915 percentage points
```

MG-CCD:

```text
Original → Counterfactual Macro-F1 drop
= 0.100 percentage points
```

MG-CCD improves counterfactual Macro F1 over the Standard RGB baseline by:

```text
12.747 percentage points
```

while ordinary RGB Macro F1 is only:

```text
0.068 percentage points lower
```

Therefore, the classification contribution is primarily **robustness-oriented**, not an ordinary-accuracy claim.

---

# Unified Research Contribution

The project connects the three tasks instead of treating them independently.

```text
PlantVillage audit + pairing
          ↓
   Segmentation study
          ↓
   validated leaf masks
        /       \
       /         \
Detection       Classification
tight boxes     mask-guided robustness
       \         /
        \       /
   unified multi-task study
```

Key links:

- segmentation QC produces reliable leaf-region supervision,
- detection converts leaf masks into tight whole-leaf boxes,
- classification uses the completed U-Net as privileged mask supervision,
- MG-CCD transfers leaf-focused teacher knowledge into an RGB-only student.

---

# Repository Structure

```text
plantvillage-multitask-vision-research/
├── README.md
├── LICENSE
├── .gitignore
│
├── notebooks/
│   ├── segmentation/
│   ├── detection/
│   └── classification/
│
├── artifacts/
│   ├── segmentation/
│   ├── detection/
│   └── classification/
│
├── results/
│   ├── segmentation/
│   ├── detection/
│   ├── classification/
│   └── overall/
│
├── docs/
│   └── notes/
│
└── archive/
    ├── detection_full_image_pseudo_box_experiments/
    └── deprecated_classification_preprocessing/
```

---

# Reproducibility

The project uses:

- fixed manifests,
- relative paths,
- fixed random seeds,
- saved configuration files,
- persistent Google Drive checkpoints,
- auto-resume logic,
- best and last checkpoints,
- CSV/JSON result artifacts,
- per-class reports,
- qualitative visualizations.

Large datasets and model checkpoints are not intended for normal Git tracking.

---

# Important Limitations

1. PlantVillage is a controlled image dataset; results should not be interpreted as direct field performance.
2. Segmentation supervision is derived from dataset-provided segmented images rather than an independent manual mask dataset.
3. Detection boxes are mask-derived whole-leaf boxes, not lesion-level annotations.
4. The PlantDoc stage has not yet been isolated with a direct COCO→PlantVillage ablation.
5. Classification background-only evaluation is diagnostic because predicted masks can leave residual leaf evidence.
6. The exact novelty wording for MG-CCD should be finalized only after the final literature review.
7. External field-domain validation would further strengthen all three task claims.

---

# Research Notes

Detailed methodology notes are stored under:

```text
docs/notes/
```

Recommended files:

```text
PlantVillage_Segmentation_Final_Research_Notes.tex
PlantVillage_Detection_Final_Research_Notes.tex
PlantVillage_Classification_Final_Research_Notes.tex
```

---

# Paper Direction

The completed experimental study contains three linked components:

```text
Segmentation:
rigorous QC + architecture comparison

Detection:
mask-derived tight leaf localization
+ plant-domain transfer learning

Classification:
segmentation-privileged
counterfactual robustness training
```

The final conference paper should emphasize the **unified methodological connection and controlled robustness analysis**, rather than simply presenting nine unrelated models.

---

## License

See the repository `LICENSE` file.

Raw third-party datasets are not redistributed by this repository.
