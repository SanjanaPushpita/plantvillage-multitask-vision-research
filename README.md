# PlantVillage Multi-Task Vision Research

Research repository for a PlantVillage-based computer-vision study covering:

- Image classification
- Leaf segmentation
- Object detection

> **Repository status:** Work in progress.  
> The segmentation preprocessing and quality-control stage is complete; model training notebooks will be added incrementally.

## Current Segmentation Status

The segmentation preprocessing pipeline has been finalized with the following verified state:

| Item | Final value |
|---|---:|
| PlantVillage classes | 38 |
| Original color images | 54,305 |
| Original grayscale images | 54,305 |
| Original segmented images | 54,306 |
| Valid color-segmented pairs after pairing | 54,305 |
| Final segmentation samples after QC | 52,969 |
| Training samples | 42,371 |
| Validation samples | 5,300 |
| Test samples | 5,298 |
| Train/validation/test overlap | 0 |
| Final target QC problems | 0 |

### Segmentation preprocessing summary

1. Audited the `color`, `grayscale`, and `segmented` directories.
2. Identified one unmatched segmented image and excluded it from the valid-pair set.
3. Built one-to-one color/segmented pairing using normalized full-name matching with descriptor fallback.
4. Created a fixed stratified 80:10:10 split using random seed 42.
5. Performed dataset quality control, including pairing checks, duplicate checks, spatial-size checks, visual verification, background-quality checks, and target validation.
6. Excluded samples without reliable foreground-background separation from the final segmentation experiment.
7. Retained all 38 PlantVillage classes.
8. Generated binary segmentation targets dynamically in memory from QC-approved segmented RGB images; no standalone binary-mask dataset is stored.
9. Final target rule: near-black background pixels are background (`0`), and retained foreground pixels are leaf (`1`), using threshold 15.
10. Verified all 52,969 final targets with no empty/effectively-full invalid targets.

## Planned Segmentation Notebooks

- `01_PlantVillage_Segmentation_Data_Preprocessing_and_QC.ipynb`
- `02_PlantVillage_UNet_Segmentation.ipynb`
- `03_PlantVillage_DeepLabV3Plus_Segmentation.ipynb`
- `04_PlantVillage_FPN_Segmentation.ipynb`
- `05_PlantVillage_Segmentation_Model_Comparison.ipynb`

Each model experiment is designed to be independent while sharing the same final manifest, preprocessing rule, split, and evaluation protocol for fair comparison.

## Suggested Repository Structure

```text
.
├── README.md
├── LICENSE
├── .gitignore
├── notebooks/
│   ├── classification/
│   ├── segmentation/
│   └── detection/
├── docs/
│   └── notes/
├── artifacts/
│   └── segmentation/
│       ├── manifests/
│       └── configs/
└── results/
    └── segmentation/
```

## Important Files to Preserve

For the segmentation experiments, the most important reproducibility artifacts are:

- `final_segmentation_manifest.csv`
- `segmentation_preprocessing_config.json`
- `segmentation_qc_exclusions.csv`
- `plantvillage_pairs_54305.csv`
- `dataset_audit.txt`

These files should be stored under `artifacts/segmentation/`.

## Dataset

The PlantVillage dataset itself is **not redistributed in this repository**. Users with authorized access should obtain the dataset from its original source and follow the relevant dataset terms.

Raw image directories should remain outside Git tracking.

## Reproducibility

The finalized segmentation experiment uses:

- fixed class-aware split
- random seed: `42`
- image size: `256 x 256`
- binary segmentation target generated dynamically
- near-black threshold: `15`
- nearest-neighbor interpolation for binary target resizing
- fixed final manifest across all segmentation architectures

## Research Notes

The preprocessing notebook contains exploratory and rejected approaches as part of the research record. In particular, an adaptive color-difference/Otsu approach was evaluated but was not selected as the final target-generation method after qualitative inspection.

The PDF methodology/worklog under `docs/notes/` is intended as an evolving personal research record rather than the final conference manuscript.

## Segmentation Progress

| Experiment | Status |
|---|---|
| Data Preprocessing & QC | ✅ Complete |
| U-Net (ResNet34) | ✅ Complete |
| DeepLabV3+ (ResNet34) | ✅ Complete |
| FPN (ResNet34) | ✅ Complete |
| Final Model Comparison | ✅ Models Completed |

### Final Segmentation Results

| Metric | U-Net | DeepLabV3+ | FPN |
|---|---:|---:|---:|
| Test Dice | **0.991319** | 0.990106 | 0.990844 |
| Test IoU | **0.982788** | 0.980405 | 0.981854 |
| Precision | **0.991544** | 0.990305 | 0.990803 |
| Recall | **0.991094** | 0.989906 | 0.990885 |
| Specificity | **0.992660** | 0.991584 | 0.992013 |
| Pixel Accuracy | **0.991933** | 0.990804 | 0.991488 |

U-Net achieved the highest observed performance on the fixed PlantVillage
segmentation test set, followed closely by FPN and DeepLabV3+.
All models used the same QC-approved segmentation dataset and ResNet34
ImageNet-pretrained encoder.

### U-Net — Final Test Performance

| Metric | Score |
|---|---:|
| Dice | 0.991319 |
| IoU / Jaccard | 0.982788 |
| Precision | 0.991544 |
| Recall | 0.991094 |
| Specificity | 0.992660 |
| Pixel Accuracy | 0.991933 |

### Segmentation Results

| Metric | U-Net | DeepLabV3+ |
|---|---:|---:|
| Dice | **0.991319** | 0.990106 |
| IoU / Jaccard | **0.982788** | 0.980405 |
| Precision | **0.991544** | 0.990305 |
| Recall | **0.991094** | 0.989906 |
| Specificity | **0.992660** | 0.991584 |
| Pixel Accuracy | **0.991933** | 0.990804 |

## Models

Segmentation models planned for controlled comparison:

1. U-Net
2. U-Net++
3. DeepLabV3+

Model checkpoints and other large generated files are intentionally excluded from normal Git tracking.

## License and Use Restrictions

This repository is proprietary research work and is **not open source**.

No permission is granted to copy, reproduce, modify, distribute, publish, sublicense, commercialize, or create derivative works from the original code, notebooks, documentation, preprocessing logic, figures, manifests, or results without prior written permission from the copyright holder.

See [LICENSE](LICENSE) for the complete terms.

## Citation

A formal citation will be added after the associated paper metadata is finalized.

## Contact

For permission requests or research correspondence:

- Name: `Maliha Sanjana`
- Email: `malihasanjanapushpita@gmail.com`
