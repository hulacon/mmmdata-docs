---
title: Output Spaces & Preprocessing Steps
parent: Data Organization
grand_parent: Dataset Description
nav_order: 5
---

# Output Spaces & Preprocessing Steps

fMRIPrep produces preprocessed BOLD data in three output spaces. Each
space applies a different chain of spatial transforms, but all share the
same upstream steps (NORDIC denoising, slice timing correction, head
motion estimation, fieldmap estimation, coregistration estimation). The
spaces differ only in **which transforms are applied at the final
resampling step**.

## Shared upstream processing

These steps are computed once per run and are identical across all output
spaces:

| Step | Tool | Description |
|------|------|-------------|
| NORDIC denoising | MATLAB `nordic_denoise.m` | Thermal noise suppression on complex-valued data, applied before fMRIPrep. Produces a denoised BOLD NIfTI in `derivatives/nordic/bids_input/`. |
| Slice timing correction | AFNI `3dTshift` | Temporal interpolation to align all slices to the middle of the TR (t = TR/2). See [STC note](#slice-timing-correction) below. |
| Head motion correction (HMC) | FSL MCFLIRT | Per-volume rigid-body (6 DOF) alignment to a reference volume (boldref). Produces `from-orig_to-boldref_desc-hmc_xfm.txt`. |
| Susceptibility distortion correction (SDC) | SDCFlows (pepolar) | B0 fieldmap estimated from AP/PA EPI pairs; stored as B-spline coefficients (`desc-coeff_fieldmap.nii.gz`). |
| Coregistration | FreeSurfer `bbregister` | Boundary-based registration of boldref to the subject's T1w anatomical. Produces `from-boldref_to-T1w_desc-coreg_xfm.txt`. |
| Spatial normalization | ANTs SyN | Non-linear warp from T1w to MNI152NLin2009cAsym template. Computed during anatomical preprocessing. |
| Confounds estimation | fMRIPrep | Framewise displacement, DVARS, CompCor components, motion parameters, cosine drift regressors. Identical across spaces. |

## Output spaces

### `func` — Native acquisition grid

| Property | Value |
|----------|-------|
| Grid | Run's original EPI grid (124 × 124 × 69) |
| Voxel size | ~1.8 × 1.8 × 2.0 mm (acquisition resolution) |
| Transforms applied | HMC + SDC |
| Transforms NOT applied | Coregistration, spatial normalization |
| STC | See [note below](#slice-timing-correction) |
| BIDS filename | `{prefix}_desc-preproc_bold.nii.gz` (no `space-` entity) |
| Primary use | Within-subject voxelwise analyses (GLMsingle), single-trial beta estimation |

The `func` space preserves the native acquisition grid with zero
interpolation-induced smoothing beyond the single HMC+SDC resampling
step. Each run has its own grid — cross-run alignment requires the
cached coregistration transforms.

### `MNI152NLin2009cAsym:res-2` — Volumetric standard space

| Property | Value |
|----------|-------|
| Grid | MNI template at 2 mm isotropic (97 × 115 × 97) |
| Voxel size | 2.0 × 2.0 × 2.0 mm |
| Transforms applied | HMC + SDC + coregistration + spatial normalization |
| STC | Yes (AFNI 3dTshift) |
| BIDS filename | `{prefix}_space-MNI152NLin2009cAsym_res-2_desc-preproc_bold.nii.gz` |
| Primary use | Group analyses, atlas-based ROI analyses, cross-subject comparisons |

All runs are in a common volumetric space. Standard neuroimaging atlases
(Schaefer, Glasser, Harvard-Oxford, etc.) apply directly without
per-subject resampling.

### `fsaverage6` — Surface standard space

| Property | Value |
|----------|-------|
| Mesh | FreeSurfer fsaverage6 (40,962 vertices per hemisphere) |
| Resolution | ~3.1 mm average vertex spacing |
| Transforms applied | HMC + SDC + coregistration + cortical surface projection |
| STC | Yes (AFNI 3dTshift) |
| BIDS filename | `{prefix}_hemi-{L,R}_space-fsaverage6_bold.func.gii` |
| Primary use | Surface analyses (ISFC, ISC, hyperalignment), cross-subject cortical patterns |

Data is sampled onto the cortical surface via FreeSurfer's
volume-to-surface projection, then resampled to the fsaverage6 template
mesh. Subcortical structures are not represented.

## Summary matrix

| | func | MNI res-2 | fsaverage6 |
|---|:---:|:---:|:---:|
| NORDIC denoising | ✓ | ✓ | ✓ |
| Slice timing correction | * | ✓ | ✓ |
| Head motion correction | ✓ | ✓ | ✓ |
| Susceptibility distortion correction | ✓ | ✓ | ✓ |
| Coregistration to T1w | — | ✓ | ✓ |
| Spatial normalization (MNI) | — | ✓ | — |
| Surface projection (fsaverage6) | — | — | ✓ |
| Interpolation steps | 1 | 1 | 1 + surface |
| Cross-run alignment | via cached xfms | intrinsic | intrinsic |
| Cross-subject alignment | no | yes | yes |

\* See [STC note](#slice-timing-correction) below.

## Slice timing correction

STC is applied in the MNI and fsaverage6 spaces (produced by fMRIPrep,
which runs AFNI `3dTshift` when `SliceTiming` metadata is present).

For the `func` space **post-hoc backfill** (existing runs resampled via
`resample_bold_to_func.py`), STC is intentionally skipped:

- GLMsingle fits per-voxel HRFs that absorb slice-timing delays
- STC's temporal interpolation can reduce tSNR for single-trial estimation
- The acquisition TR (1.5 s) is short enough that the HRF shape change
  across slices is modest relative to per-voxel HRF fitting flexibility

For **future fMRIPrep runs** (new subjects/sessions), fMRIPrep applies
STC to all output spaces including `func`. To produce non-STC'd func
output for new data, either use `--ignore slicetiming` or run the
post-hoc script.

**Follow-up:** An empirical comparison of GLMsingle voxel reliability
with and without STC is planned to validate this decision.

## Per-run transform files

Each BOLD run in `fmriprep_nordic/{sub}/{ses}/func/` includes cached
transforms that can be used to move between spaces:

| File | Description |
|------|-------------|
| `*_from-orig_to-boldref_desc-hmc_xfm.txt` | Per-volume rigid HMC transforms (ITK format, one affine per volume) |
| `*_from-boldref_to-B0map*_xfm.txt` | Rigid alignment from boldref to fieldmap space |
| `*_from-boldref_to-T1w_desc-coreg_xfm.txt` | Boundary-based coregistration (boldref → T1w) |
| `*_desc-hmc_boldref.nii.gz` | Motion-corrected reference volume |
| `*_desc-coreg_boldref.nii.gz` | Coregistered reference volume |

Fieldmap coefficients are in `fmriprep_nordic/{sub}/{ses}/fmap/`:

| File | Description |
|------|-------------|
| `*_fmapid-*_desc-coeff_fieldmap.nii.gz` | B-spline SDC coefficients |
| `*_fmapid-*_desc-epi_fieldmap.nii.gz` | EPI reference aligned with coefficients |
| `*_fmapid-*_desc-preproc_fieldmap.nii.gz` | Preprocessed fieldmap in Hz (for visualization) |

Two fieldmaps per session: one for encoding runs (`B0mapencodingXXX`),
one for retrieval/resting runs (`B0mapretrievalXXX`). The mapping from
BOLD run to fieldmap is encoded in the `from-boldref_to-B0map*` filename.

The T1w → MNI warp is computed during anatomical preprocessing and
stored in `fmriprep_nordic/{sub}/anat/`.

## Fieldmap-to-run mapping

Each session acquires two pairs of AP/PA EPI fieldmaps:

| Fieldmap ID pattern | Applies to |
|---------------------|------------|
| `B0mapencodingXXXXX` | TBencoding (all runs), TBmath |
| `B0mapretrievalXXXX` | TBretrieval (all runs), TBresting |

The fieldmap ID is embedded in both the transform filename
(`from-boldref_to-B0map{id}_xfm.txt`) and the coefficient filename
(`fmapid-B0map{id}_desc-coeff_fieldmap.nii.gz`), so the mapping is
self-documenting.
