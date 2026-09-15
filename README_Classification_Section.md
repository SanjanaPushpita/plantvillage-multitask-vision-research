## Classification — Background-Robust Disease Classification

### Research Objective

The classification study was designed to move beyond a conventional PlantVillage benchmark in which several CNNs are trained and compared only by ordinary test accuracy.

The final question is:

> Can a classifier preserve near-perfect ordinary PlantVillage classification while remaining stable when the image background is deliberately changed?

### Final Classification Dataset

The classification component uses the paired PlantVillage dataset after exclusion of one unusable source sample.

| Item | Count |
|---|---:|
| Total | 54,304 |
| Train | 43,443 |
| Validation | 5,430 |
| Test | 5,431 |
| Classes | 38 |

The original fixed stratified 80:10:10 split with seed 42 is preserved.

### Why Classification Uses 54,304 Samples

The final segmentation subset contains 52,969 images because segmentation-specific source-quality filtering removed segmented images with unreliable explicit foreground/background separation.

Reusing that subset for classification caused one class (`Corn_(maize)___healthy`) to have zero test samples. The classification source was therefore corrected to the earlier 54,304-sample paired dataset.

The completed U-Net segmentation model was then used to generate a leaf mask for every classification image.

### Four Evaluation Views

```text
Original RGB
Leaf-only
Background-only
Counterfactual background
```

The counterfactual view keeps the target leaf while changing only its background using a same-split, different-class donor image.

### Classification Experiments

#### 1. Standard RGB EfficientNet-B0

Conventional RGB classification baseline.

#### 2. Leaf-Only EfficientNet-B0 Teacher

Trained on U-Net-derived leaf-only images and used as privileged training supervision.

#### 3. MG-CCD EfficientNet-B0

**Mask-Guided Counterfactual Consistency Distillation (MG-CCD)** is the proposed robustness-oriented training strategy.

The student is trained with:

```text
Original RGB cross-entropy
+ Counterfactual cross-entropy
+ Leaf-only teacher distillation
+ Original/Counterfactual consistency loss
```

At inference time, MG-CCD requires **only an ordinary RGB image**.

### Final Multi-View Macro-F1 Results

| Model | Original | Leaf-only | Counterfactual | Background-only |
|---|---:|---:|---:|---:|
| Standard RGB | **0.997589** | 0.648865 | 0.868435 | 0.040040 |
| Leaf-Only Teacher | 0.950641 | **0.996449** | 0.957307 | 0.080980 |
| MG-CCD | 0.996907 | 0.986897 | **0.995904** | 0.088776 |

### Main Robustness Finding

Standard RGB:

```text
Original Macro F1       = 0.997589
Counterfactual Macro F1 = 0.868435
Drop                    = 0.129153
                          = 12.915 percentage points
```

MG-CCD:

```text
Original Macro F1       = 0.996907
Counterfactual Macro F1 = 0.995904
Drop                    = 0.001002
                          = 0.100 percentage points
```

MG-CCD improves counterfactual Macro F1 over the Standard RGB baseline by:

```text
0.127469
= 12.747 percentage points
```

while ordinary RGB Macro F1 changes by only:

```text
-0.000682
= -0.068 percentage points
```

### Interpretation

The proposed model does **not** achieve the highest ordinary RGB Macro F1; the Standard RGB baseline is marginally higher.

The contribution is that MG-CCD preserves near-baseline ordinary RGB classification while strongly improving robustness to the controlled background intervention and substantially improving leaf-only performance.

### Important Limitation

`background_only` is treated as a diagnostic rather than a pure shortcut-learning metric because predicted masks are not perfect and may leave residual leaf pixels.

The stronger robustness evidence is the controlled **Original vs Counterfactual** comparison.

### Reproducibility

All training notebooks save:

```text
best_model.pth
last_checkpoint.pth
```

to Google Drive.

Interrupted Colab runs can resume from the last completed epoch.

The common project root is:

```text
/content/drive/MyDrive/PlantVillage_Classification
```

### Final Classification Notebooks

```text
01_PlantVillage_Classification_Data_Preparation_and_Bias_Audit_FINAL.ipynb
01B_Classification_U-Net_Mask_Intervention_QC.ipynb
02_PlantVillage_Standard_RGB_EfficientNetB0.ipynb
03_PlantVillage_Leaf_Only_Teacher_EfficientNetB0.ipynb
04_PlantVillage_Mask_Guided_Counterfactual_Consistency_Distillation.ipynb
05_PlantVillage_Classification_Robustness_and_Model_Comparison.ipynb
```

### Novelty Caution

The exact novelty wording for MG-CCD should be finalized only after a systematic literature review. The repository describes it as the **proposed training strategy used in this study**, not as a verified first-ever method.
