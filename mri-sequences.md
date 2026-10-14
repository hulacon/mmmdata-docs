---
title: MRI Sequences
parent: Data Acquisition
grand_parent: Dataset Description
nav_order: 1
---

# MRI Sequence Information

All data were acquired on a **Siemens MAGNETOM Prisma 3T** scanner
(software: syngo MR XA30 for the first three participants, XA60 from the
fourth on) at the University of Oregon, using a
**64-channel head/neck coil** (HeadNeck_64).

## Structural Data

### T1-weighted (MPRAGE)

| Parameter | Value |
|-----------|-------|
| Protocol | ABCD_T1w_MPR_vNav |
| Sequence | 3D GR/IR (tfl_mgh_epinav_ABCD) |
| TR | 2500 ms |
| TE | 2.9 ms |
| TI | 1070 ms |
| Flip angle | 8° |
| Voxel size | 1.0 mm isotropic |
| Matrix | 256 × 256 × 176 |
| Acceleration | GRAPPA 2× |
| Sessions | ses-01, ses-28, ses-30 |

### T2-weighted (SPACE)

| Parameter | Value |
|-----------|-------|
| Protocol | ABCD_T2w_SPC_vNav |
| Sequence | 3D SE (space_mgh_epinav_ABCD) |
| TR | 3200 ms |
| TE | 565 ms |
| Flip angle | 120° (variable) |
| Voxel size | 1.0 mm isotropic |
| Matrix | 256 × 256 × 176 |
| Acceleration | GRAPPA 2× |
| Echo train length | 190 |
| Sessions | ses-01 |

### T2-weighted oblique coronal

| Parameter | Value |
|-----------|-------|
| Protocol | T2_coronal_1.8 |
| Sequence | 2D TSE |
| TR | 11250 ms |
| TE | 84 ms |
| Flip angle | 150° |
| Voxel size | 0.43 × 0.43 × 1.8 mm (512 × 512 × 50) |
| Averages | 2 |
| Acceleration | GRAPPA 2× |
| Orientation | Oblique coronal (hippocampal long axis) |
| Sessions | ses-01 |

## Diffusion Data

| Parameter | Value |
|-----------|-------|
| Protocol | cmrr_diff_3shell |
| Sequence | 2D EPI (cmrr_mbep2d_diff) |
| TR | 3000 ms |
| TE | 70 ms |
| Flip angle | 90° |
| Voxel size | 1.8 mm isotropic |
| Matrix | 116 × 116 |
| Multiband factor | 3 |
| Acceleration | GRAPPA 2× |
| Partial Fourier | 6/8 |
| Diffusion scheme | Monopolar, 3-shell |
| Phase encoding dirs | AP, PA, LR, RL |
| Total readout time | 42.0 ms |
| Sessions | ses-01, ses-28 |

## Functional Data

### BOLD (all functional tasks)

| Parameter | Value |
|-----------|-------|
| Protocol | Resting_baseline / task-specific |
| Sequence | 2D EPI (cmrr_mbep2d_bold) |
| TR | 1500 ms |
| TE | 29 ms |
| Flip angle | 90° |
| Voxel size | 1.8 × 1.8 × 2.0 mm |
| Matrix | 124 × 124 |
| Slices | 69 (23 per MB group × 3) |
| Multiband factor | 3 |
| Acceleration | GRAPPA 2× |
| Phase encoding | AP (j−) |
| Total readout time | 35.05 ms |
| Effective echo spacing | 0.285 ms |

All functional tasks use identical acquisition parameters, with one
exception: the first participant's first two sessions were acquired at
1.7 mm isotropic (total readout time 36.3 ms, effective echo spacing
0.295 ms) before the protocol settled on the values above. Task-specific
protocol names vary but the underlying sequence is the same.

## Fieldmaps

### Spin-echo EPI (for distortion correction)

| Parameter | Value |
|-----------|-------|
| Protocol | se_epi_ap / se_epi_pa |
| Sequence | 2D SE-EPI (cmrr_mbep2d_se) |
| TR | 6500 ms |
| TE | 40.6 ms |
| Flip angle | 90° |
| Voxel size | 1.7 mm isotropic |
| Matrix | 124 × 124 |
| Phase encoding dirs | AP and PA (paired) |
| Total readout time | 36.3 ms |
| Effective echo spacing | 0.295 ms |
| Sessions | All functional sessions from ses-02 (from ses-01 in later participants) |

AP/PA pairs are acquired at the start of each functional session for
TOPUP-based distortion correction. The fieldmaps keep the 1.7 mm isotropic
grid and its 36.3 ms readout, so they match the BOLD readout only in the
first participant's first two sessions; elsewhere the BOLD readout is
35.05 ms.
