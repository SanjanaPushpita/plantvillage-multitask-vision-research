# Repository Setup Notes

## Recommended repository name

**Primary recommendation:** `plantvillage-multitask-vision-research`

Other good options:

- `plantvillage-classification-segmentation-detection`
- `plantvillage-unified-computer-vision`
- `plantvillage-multitask-deep-learning`

The first option is recommended because it is short enough for GitHub while still covering all three tasks.

## Recommended visibility

Create the repository as **Private** while the research is ongoing.

If the repository is later made public, review the repository license, paper publication status, third-party dataset terms, and any unpublished experimental details before changing visibility.

## First files to add

1. `README.md`
2. `LICENSE`
3. `.gitignore`
4. `notebooks/segmentation/01_PlantVillage_Segmentation_Data_Preprocessing_and_QC.ipynb`
5. `docs/notes/segmentation_preprocessing_worklog.pdf`
6. reproducibility artifacts under `artifacts/segmentation/`

## Files that should not be committed

- Raw PlantVillage images
- Kaggle credentials
- Google Drive credentials
- Large model checkpoints
- Temporary prediction images
- Colab/Jupyter checkpoint folders
- Environment folders

## Recommended commit sequence

```text
chore: initialize research repository
docs: add segmentation preprocessing README and notes
feat: add segmentation preprocessing and QC notebook
data: add segmentation manifests and preprocessing config
feat: add U-Net segmentation experiment
feat: add U-Net++ segmentation experiment
feat: add DeepLabV3+ segmentation experiment
analysis: add segmentation model comparison
```

## Notebook policy

Keep one notebook per major experiment.

The preprocessing notebook should remain frozen after the final preprocessing state is accepted. Model notebooks should load the saved manifest/config instead of repeating the preprocessing pipeline.

For GitHub readability, prefer a notebook copy with outputs cleared. Preserve the full output-heavy notebook separately in Google Drive as the archival research record.
