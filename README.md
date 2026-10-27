# hecktor-nnunet

An end-to-end [nnU-Net v2](https://github.com/MIC-DKFZ/nnUNet) pipeline for segmenting head and neck tumors on PET/CT scans from the **HECKTOR 2026** challenge, built to run on Google Colab.

The model segments two targets:

- **GTVp**: the primary tumor
- **GTVn**: metastatic lymph nodes

This work was done as part of a team project (team repository: [ggwp2333/HECKTOR_2026](https://github.com/ggwp2333/HECKTOR_2026)). This repository contains my training pipeline.

## Pipeline

The notebook [`HECKTOR_nnUNet_train.ipynb`](HECKTOR_nnUNet_train.ipynb) walks through every step. Cell-by-cell explanations are written in Chinese.

| Step | What it does |
| --- | --- |
| 1 | Install `nnunetv2` and check the GPU |
| 2 | Mount Google Drive and set the `nnUNet_raw`, `nnUNet_preprocessed` and `nnUNet_results` paths; results are saved to Drive so they survive Colab disconnects |
| 3 | Select patients that have CT, PET and labels; exclude very large scans (> 350 slices) that exceed Colab's RAM; sample 50 with a fixed seed |
| 4 | Convert to nnU-Net format: CT as channel `_0000`, PET as channel `_0001` |
| 5 | Write `dataset.json` (0 = background, 1 = GTVp, 2 = GTVn) and spot-check shapes and label values |
| 6a | Run nnU-Net experiment planning only (`--no_pp`) |
| 6b | Preprocess one case at a time with resume support, so a crash on one oversized case does not lose earlier work |
| 7 | Train `3d_fullres`, fold 0 (40 train / 10 validation), with `nnUNetTrainer_50epochs`; resume with `--c` after a disconnect |
| 8 | Show the training curve and validation Dice for GTVp and GTVn |

### Engineering notes

The default nnU-Net preprocessing runs several worker processes and was repeatedly killed for running out of memory on free Colab (~12 GB RAM). Splitting planning from preprocessing and processing cases sequentially fixed this. Partially written outputs from a crashed case are cleaned up and the case is added to a skip list, so a bad file never reaches training. Data-augmentation workers are also limited to 2 during training.

## How to run

1. Open the notebook in Colab and set the runtime to GPU (a T4 is enough; 50 epochs take about 5–6 hours).
2. Add the shared `HECKTOR_2026` data folder to your own Google Drive.
3. Run the cells from top to bottom.

On a machine with more memory, raise or remove `MAX_SLICES`, and step 6 can be replaced by a single `nnUNetv2_plan_and_preprocess` call.

## Data

The HECKTOR 2026 data must be requested from the challenge organizers. **No imaging data is included in this repository.**

## Limitations

- Results are preliminary: one fold, 50 epochs, 50 patients.
- After excluding large scans, most sampled patients come from a single center (MDA), so the training set is not balanced across hospitals.
