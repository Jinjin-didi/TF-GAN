# TF-GAN

**A time–frequency-guided generative adversarial network for few-window fault diagnosis in marine observation networks**

Ruixin Chen, Rufu Qin, Yang Yu

> **Repository status: under preparation.**
> This repository currently contains this README only. The code, the configuration files of
> every reported run, and the Xiaoqushan temperature dataset will be uploaded upon acceptance
> of the manuscript. An anonymised copy of the code and data is available to reviewers on
> request during review.

---

## 1. What this repository will contain

| Path | Contents |
|---|---|
| `tfgan/` | TF-GAN: conditional generator with multi-head self-attention, multi-period discriminator (periods 2, 3, 5), zero-padded translation, three-resolution STFT spectral loss, support-anchoring operator |
| `baselines/` | SMOTE, ADASYN, Borderline-SMOTE, jitter, shift, within-class interpolation (WCI), class-weighted cross-entropy, focal loss, class-balanced loss, DDPM, convolutional VAE, Transformer-GAN, DWT-MWCGAN, DCGAN, WGAN-GP |
| `classifier/` | The 1-D CNN used as the downstream diagnostic model, with the six classifier seeds (2–7) |
| `data/xqs_temperature/` | Xiaoqushan temperature substrate windows, the window index file, and SHA-256 hashes of the training, validation and test partitions |
| `injection/` | Standards-calibrated fault injection code (QARTOD Test 6/7/8 thresholds and gross-range bounds) and the per-window manifest recording every injected parameter |
| `configs/` | The `setting.txt` of every reported run, one file per run |
| `scripts/` | Entry points that regenerate each table and figure of the paper |

## 2. What is **not** in this repository

The current-signal field record was acquired on a production asset of a marine observation
network and is subject to that network's data-sharing agreement. It is not redistributed here.
Requests for access are considered by the corresponding author and are subject to the same
agreement. No result in the paper depends on redistributing that record.

## 3. Reproducing the results (available on acceptance)

Environment used for every reported run:

```
Python 3.10.18
PyTorch 2.8.0 + CUDA 12.8
One NVIDIA RTX 4080 (16 GB)
```

Training used 10,000 epochs, batch size 10, latent dimension 10, and six downstream
classifier seeds. The peak GPU memory of one TF-GAN training run with the spectral term
active is 2.5–3.5 GB, so two runs can be launched in parallel on a 16 GB card.

```bash
# 1. build the injection benchmark
python injection/inject_std_calibrated.py --config configs/xqs_temperature_severe.yaml

# 2. train TF-GAN for one labelled-window budget (K = 5 shown)
python tfgan/train.py --config configs/tfgan_xqs_k5.yaml --seed 21

# 3. generate and anchor
python tfgan/generate.py --ckpt runs/tfgan_xqs_k5/checkpoints/best.ckpt \
                         --per-class 600 --anchor-lo 0.10 --anchor-hi 0.95

# 4. train the classifier and evaluate
python classifier/train_eval.py --generated runs/tfgan_xqs_k5/generated.pt --seed 2

# 5. regenerate one table or figure
python scripts/table9_main_comparison.py
python scripts/fig12_stft_panels.py
```

`scripts/` maps one file to each table and figure of the paper, so a reported number can be
traced from the manuscript to a single command.

## 4. Data

| Dataset | Availability |
|---|---|
| Xiaoqushan temperature substrate (used for calibrated injection) | released in `data/xqs_temperature/` on acceptance, with window index and partition hashes |
| Current-signal field record (naturally occurring faults) | held by the observing network; available from the corresponding author with the network's permission |
| Public records cited in the paper | obtained from their original providers under their own licences; not redistributed here |

## 5. Intended use

The code is released for research on data augmentation for one-dimensional sensor fault
diagnosis. It is not a certified diagnostic system, and no output of this repository should be
used to make maintenance or safety decisions on an operating asset without independent
validation.

## 6. Citation

The manuscript is under review. Until it is published, please cite it as:

```
Chen, R., Qin, R., Yu, Y. TF-GAN: a time–frequency-guided generative adversarial network for
few-window fault diagnosis in marine observation networks. Manuscript under review (2026).
```

A BibTeX entry with the final volume and DOI will be added on acceptance.

## 7. Contact

Corresponding author: Rufu Qin. For questions about the code, the data, or reviewer access,
please open an issue in this repository or contact the corresponding author directly.
