---
title: Quality Assurance
parent: Quality & Compliance
grand_parent: Dataset Description
nav_order: 1
---

# Quality Assurance

This page documents the standard operating procedure (SOP) for image quality
assurance (QA) of the MMMData dataset. The workflow draws on recommendations
from the neuroimaging QA literature (Liu, 2016; Provins et al., 2023;
Esteban et al., 2017, 2019) and uses two automated tools already integrated
into the processing pipeline:

| Tool | Version | Purpose |
|------|---------|---------|
| **MRIQC** | v24.1.0 | Computes image quality metrics (IQMs) and generates visual reports for structural and functional scans |
| **fMRIPrep** | v24.1.1 | Produces confound regressors including framewise displacement, DVARS, and CompCor components |

---

## Overview of QA Stages

```
BIDS raw
  │
  ├──▶ Stage 1 — Automated IQM extraction (MRIQC)
  │
  ├──▶ Stage 2 — Visual inspection of MRIQC reports
  │
  ├──▶ Stage 3 — Motion & confound assessment (fMRIPrep)
  │
  ├──▶ Stage 4 — Outlier detection & run-level decisions
  │
  └──▶ Stage 5 — Reporting & documentation
```

---

## Stage 1: Automated Image Quality Metrics (MRIQC)

MRIQC computes per-image IQMs from the raw BIDS data, without any
preprocessing beyond basic intensity normalization. Metrics are stored as
JSON sidecar files alongside HTML visual reports in `derivatives/mriqc/`.

### Key Structural IQMs (T1w / T2w)

| Metric | Description | Interpretation |
|--------|-------------|----------------|
| `cnr` | Contrast-to-noise ratio | Higher = better GM/WM contrast; sensitive to coil inhomogeneity |
| `cjv` | Coefficient of joint variation | Lower = better tissue separation; high values suggest motion or artifact |
| `snr_total` | Signal-to-noise ratio (total) | Higher = better; summary of overall image quality |
| `snr_gm` / `snr_wm` | SNR within GM / WM masks | Tissue-specific signal quality |
| `efc` | Entropy focus criterion | Lower = sharper image; high values indicate blurring or ghosting |
| `fber` | Foreground-to-background energy ratio | Higher = better; low values suggest head-coil or wrap-around issues |
| `qi_1` | Quality index 1 (artifact detection) | Proportion of voxels in artifact-heavy regions; lower is better |
| `qi_2` | Quality index 2 (noise distribution) | Goodness-of-fit of noise model; lower is better |
| `fwhm_avg` | Full-width half-maximum (smoothness) | Expected range depends on voxel size; deviations suggest motion |
| `inu_range` / `inu_med` | Intensity non-uniformity range / median | Larger values indicate worse bias field |
| `tpm_overlap_gm` / `_wm` / `_csf` | Tissue probability map overlap | Higher = better spatial normalization; low overlap flags registration failure |
| `wm2max` | White-matter to maximum intensity ratio | Helps detect intensity clipping |

### Key Functional IQMs (BOLD)

| Metric | Description | Interpretation |
|--------|-------------|----------------|
| `tsnr` | Temporal signal-to-noise ratio | Higher = better; reflects system stability over the run |
| `snr` | Signal-to-noise ratio | Overall image quality |
| `fd_mean` | Mean framewise displacement | Lower = less motion; primary motion summary statistic |
| `fd_num` | Number of high-FD volumes (> 0.2 mm) | Count of motion-contaminated volumes |
| `fd_perc` | Percentage of high-FD volumes | Proportion of run affected by motion |
| `dvars_std` / `dvars_nstd` | Standardized / non-standardized DVARS | Reflects volume-to-volume signal change; spikes indicate motion or artifact |
| `efc` | Entropy focus criterion | Image sharpness / ghosting |
| `fber` | Foreground-to-background energy ratio | Signal quality |
| `gcor` | Global correlation | Degree of global signal fluctuation; very high values suggest physiological or hardware artifact |
| `aqi` | Artifact quality index | Lower = better; summary artifact metric |
| `aor` | AFNI outlier ratio | Proportion of outlier voxels across volumes |
| `fwhm_avg` | Smoothness estimate | Should be consistent across runs of the same sequence |
| `gsr_x` / `gsr_y` | Ghost-to-signal ratio (x / y) | Quantifies N/2 ghosting in each phase-encode direction |

---

## Stage 2: Visual Inspection of MRIQC Reports

MRIQC generates per-image HTML reports containing diagnostic visualizations.
The following should be reviewed for every acquired scan:

### Structural (T1w / T2w)

1. **Overall image quality**: Check for gross motion artifact (ringing),
   signal dropout, or wrap-around
2. **Tissue segmentation overlay**: Verify that GM/WM/CSF boundaries are
   anatomically plausible
3. **Background noise pattern**: Look for structured noise (RF interference)
   vs. expected thermal noise
4. **Intensity inhomogeneity map**: Confirm bias field correction is reasonable

### Functional (BOLD)

1. **Mean BOLD image**: Check for signal dropout in susceptibility-prone
   regions (orbitofrontal, inferior temporal, medial temporal lobe)
2. **Temporal standard deviation map**: High-variance regions should correspond
   to vasculature / CSF; cortical hotspots suggest uncorrected motion
3. **Carpet plot (grayplot)**: Look for vertical stripes (global signal
   shifts) and banding patterns (respiration, motion); a clean carpet plot
   shows largely homogeneous signal
4. **FD trace**: Identify periods of excessive motion; correlate with task
   timing to assess whether motion is task-correlated
5. **DVARS trace**: Should track FD; discrepancies may indicate non-motion
   artifacts

---

## Stage 3: Motion and Confound Assessment (fMRIPrep)

After preprocessing, fMRIPrep writes per-run confound timeseries files
(`*_desc-confounds_timeseries.tsv`) containing the following motion-related
columns used for QA:

| Column | Description |
|--------|-------------|
| `framewise_displacement` | Head displacement (mm) between consecutive volumes (Power et al., 2012) |
| `dvars` | Root-mean-square intensity change between consecutive volumes |
| `std_dvars` | Standardized DVARS (normalized by temporal mean) |
| `rmsd` | Root-mean-square deviation of head-realignment parameters |
| `trans_x`, `trans_y`, `trans_z` | Translation parameters (mm) |
| `rot_x`, `rot_y`, `rot_z` | Rotation parameters (radians) |

### Motion Summary Statistics

For each BOLD run, compute:

- **Mean FD**: Primary motion summary; threshold for concern varies by study
  (see below)
- **Max FD**: Identifies single worst motion event
- **Proportion of high-FD volumes**: Percentage of volumes exceeding a
  specified FD threshold
- **Mean DVARS**: Overall signal instability

These summaries are produced by the `summarize_motion()` function in the
project's `qc.py` module.

---

## Stage 4: Outlier Detection and Run-Level Decisions

### Candidate Thresholds

The following thresholds are drawn from the literature and should be
considered starting points. **Final thresholds for MMMData are TBD and
should be determined based on the empirical distribution of IQMs across
this dataset.**

| Criterion | Conservative | Moderate | Lenient | Source |
|-----------|-------------|----------|---------|--------|
| Mean FD | > 0.2 mm | > 0.5 mm | > 0.9 mm | Power et al. (2012, 2014) |
| Percentage of high-FD volumes (FD > 0.2 mm) | > 20% | > 30% | > 50% | Parkes et al. (2018) |
| Max translation | > 0.5 mm | > 1.0 mm | > 1.5 mm | Liu (2016) |
| Max rotation | > 0.5° | > 1.0° | > 1.5° | Liu (2016) |
| tSNR | < dataset 5th percentile | — | — | Empirical |
| DVARS spikes | — | — | — | Flagged by IQR-based detection |

### Statistical Outlier Detection

The project's `detect_outliers()` function flags runs whose IQMs fall
outside the interquartile range (IQR) fence:

- **Lower fence**: Q1 − _k_ × IQR
- **Upper fence**: Q3 + _k_ × IQR

where _k_ = 1.5 (standard) or 3.0 (extreme-only). Outlier detection can be
applied in two scopes:

- **Global**: Thresholds computed across all runs in the dataset
- **Within-subject**: Thresholds computed per participant, useful for
  identifying session-specific problems in a longitudinal design

### Decision Framework

Run-level disposition should follow a tiered approach:

1. **Include**: Run meets all quality thresholds
2. **Flag for review**: Run exceeds one or more soft thresholds; visual
   inspection determines inclusion
3. **Include with censoring**: Run has isolated high-motion volumes that can
   be censored (scrubbed) during analysis; overall run quality is acceptable
4. **Exclude**: Run exceeds hard thresholds or visual inspection reveals
   uncorrectable artifact

> **Note**: In a longitudinal, densely-sampled design like MMMData,
> within-subject outlier detection is particularly important because
> between-subject comparisons of absolute IQM values are less meaningful
> than detecting sessions where a participant's data quality deviates from
> their own baseline.

---

## Stage 5: Reporting and Documentation

QA outcomes should be documented at two levels:

### Run-Level QA Log

A structured record (TSV or database) for each functional and structural
run with columns:

| Field | Description |
|-------|-------------|
| `subject` | Subject ID |
| `session` | Session ID |
| `task` | Task label |
| `run` | Run number |
| `modality` | `bold`, `T1w`, `T2w`, `dwi` |
| `disposition` | `include` / `flag` / `censor` / `exclude` |
| `reason` | Free-text reason for non-include dispositions |
| `mean_fd` | Mean framewise displacement (BOLD only) |
| `pct_high_fd` | Percentage of volumes with FD > threshold |
| `tsnr` | Temporal SNR (BOLD only) |
| `reviewer` | Initials of reviewer |
| `date` | Date of review |

### Dataset-Level QA Summary

Aggregate statistics for reporting in publications:

- Number of runs/participants excluded at each stage
- Distribution plots of key IQMs (FD, tSNR, CNR) across participants and
  sessions
- Correlation between motion and task variables (to assess task-correlated
  motion)

---

## Programmatic QA Tools

The project provides a Python QC library (`neuroimaging.qc`) with functions
for programmatic QA:

| Function | Purpose |
|----------|---------|
| `get_iqm_table()` | Extract per-run MRIQC IQMs into a structured table |
| `aggregate_iqms()` | Compute summary statistics grouped by subject, session, or task |
| `detect_outliers()` | Flag runs outside IQR-based fences (global or within-subject scope) |
| `summarize_motion()` | Summarize fMRIPrep motion confounds across BOLD runs |
| `run_details()` | Deep-dive into a specific run's IQMs and motion profile |
| `processing_status()` | Verify completeness of MRIQC/fMRIPrep processing |

Usage example:

```python
from neuroimaging.qc import detect_outliers, summarize_motion

# Flag BOLD runs with outlier IQMs across the dataset
outliers = detect_outliers("derivatives/mriqc", modality="bold", scope="global")

# Summarize motion for a specific subject
motion = summarize_motion("derivatives/fmriprep", subject="03", fd_threshold=0.5)
```

---

## References

- Esteban, O., Birman, D., Schaer, M., Koyejo, O. O., Poldrack, R. A., &
  Gorgolewski, K. J. (2017). MRIQC: Advancing the automatic prediction of
  image quality in MRI from unseen sites. *PLoS ONE*, 12(9), e0184661.
- Esteban, O., Markiewicz, C. J., Blair, R. W., et al. (2019). fMRIPrep: a
  robust preprocessing pipeline for functional MRI. *Nature Methods*, 16(1),
  111–116.
- Liu, T. T. (2016). Noise contributions to the fMRI signal: An overview.
  *NeuroImage*, 143, 141–151.
- Parkes, L., Fulcher, B., Yücel, M., & Fornito, A. (2018). An evaluation
  of the efficacy, reliability, and sensitivity of motion correction
  strategies for resting-state functional MRI. *NeuroImage*, 171, 415–436.
- Power, J. D., Barnes, K. A., Snyder, A. Z., Schlaggar, B. L., & Petersen,
  S. E. (2012). Spurious but systematic correlations in functional
  connectivity MRI networks arise from subject motion. *NeuroImage*, 59(3),
  2142–2154.
- Power, J. D., Mitra, A., Laumann, T. O., Snyder, A. Z., Schlaggar, B. L.,
  & Petersen, S. E. (2014). Methods to detect, characterize, and remove
  motion artifact in resting state fMRI. *NeuroImage*, 84, 320–341.
- Provins, C., Lajous, H., Savary, E., et al. (2023). Quality control in
  functional MRI studies with MRIQC and fMRIPrep. *Frontiers in
  Neuroimaging*, 1, 1073734.
