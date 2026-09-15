## Detection — Leaf-Level Disease Detection

### Scope

The detection component performs **whole-leaf localization with PlantVillage disease-class prediction**. It is **not lesion-level detection**.

PlantVillage does not provide object-detection bounding boxes. Therefore, the final detection dataset was constructed from the QC-approved segmented leaf representations used in the segmentation study.

### Detection Dataset Preparation

The final detection pipeline was created as follows:

1. Reuse the final QC-approved PlantVillage segmentation manifest.
2. Generate the finalized binary leaf foreground mask using the retained segmented RGB images and threshold 15.
3. Compute a tight axis-aligned bounding rectangle around the foreground pixels.
4. Convert each bounding rectangle to normalized YOLO `class_id x_center y_center width height` format.
5. Preserve the existing fixed train/validation/test split.
6. Run numerical bounding-box QC.
7. Flag extreme/suspicious boxes for manual review.
8. Review 15 suspicious candidates:
   - 5 genuine small/narrow leaves were retained.
   - 10 samples were excluded because the mask-derived box covered only a small part of a visibly larger leaf.
9. Freeze the final detection manifest.

Final detection dataset:

| Item | Count |
|---|---:|
| Total samples | 52,959 |
| Training | 42,363 |
| Validation | 5,300 |
| Test | 5,296 |
| Classes | 38 |
| Manually excluded during detection QC | 10 |

### Transfer-Learning Protocol

All three final models use the same domain-transfer pipeline:

```text
COCO-pretrained detector
        ↓
PlantDoc object-detection training
        ↓
Best PlantDoc checkpoint
        ↓
PlantVillage mask-derived tight leaf boxes
        ↓
Final PlantVillage fine-tuning
        ↓
Fixed PlantVillage test evaluation
```

PlantDoc is used as an intermediate plant-domain detection stage. The current experiments do not include a direct COCO→PlantVillage ablation, so the study does not claim that PlantDoc transfer alone caused the final performance gain.

Common settings:

| Setting | Value |
|---|---|
| Image size | 320 |
| Batch size | 16 |
| Seed | 42 |
| Optimizer | AdamW |
| Initial learning rate | 0.001 |
| Weight decay | 0.0005 |
| PlantDoc max epochs | 15 |
| PlantDoc patience | 5 |
| PlantVillage max epochs | 20 |
| PlantVillage patience | 7 |
| Mixed precision | Enabled |
| Persistent recovery | Google Drive `last.pt` |

### Final Test Results

| Model | Precision | Recall | F1 | mAP50 | mAP50-95 |
|---|---:|---:|---:|---:|---:|
| YOLOv8n | 0.992832 | 0.991846 | 0.992339 | 0.994014 | 0.981625 |
| **YOLOv8s** | **0.994737** | **0.995709** | **0.995223** | **0.994231** | **0.983002** |
| YOLO11n | 0.993128 | 0.991207 | 0.992167 | 0.993787 | 0.978832 |

**Best observed model:** YOLOv8s, using mAP50-95 as the primary ranking metric.

The numerical differences are small, so the results are reported as an observed ranking on the controlled PlantVillage evaluation set rather than evidence of universal model superiority.

### Reproducibility and Runtime Recovery

Each training notebook writes checkpoints directly to Google Drive. If Colab disconnects or power is lost, the notebook detects `last.pt` and resumes the interrupted stage. Completed stages contain `_STAGE_COMPLETE.json` and are skipped on the next `Run all`.

The workflow is portable across Google accounts if the same Drive path is preserved:

```text
/content/drive/MyDrive/PlantVillage_Detection
```

For an interrupted model, copy the model's complete `experiments/<MODEL>/` directory so that `last.pt` is also available.

### Detection Notebooks

```text
01_PlantVillage_Detection_Data_Preprocessing_and_QC.ipynb
02_PlantVillage_YOLOv8n_Transfer_Detection.ipynb
03_PlantVillage_YOLOv8s_Transfer_Detection.ipynb
04_PlantVillage_YOLO11n_Transfer_Detection.ipynb
05_PlantVillage_Detection_Model_Comparison.ipynb
```

### Important Limitation

The bounding boxes are **mask-derived whole-leaf boxes**, not manually annotated lesion boxes. Therefore, the detection claim is limited to whole-leaf localization plus disease-class prediction. Performance on unconstrained field imagery should not be inferred from PlantVillage results alone.
