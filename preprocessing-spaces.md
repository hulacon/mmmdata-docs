---
title: Output Spaces & Preprocessing Steps
parent: Data Organization
grand_parent: Dataset Description
nav_order: 5
---

# Output Spaces & Preprocessing Steps

`derivatives/fmriprep/` provides five output spaces: `func`
(native acquisition grid), `T1w`, `MNI152NLin2009cAsym res-2`, `fsnative`
and `fsaverage6`. The catalog records which runs each space covers.

This page describes five output spaces. Each
space applies a different chain of spatial transforms, but all share the
same upstream steps (slice timing correction, head motion estimation,
fieldmap estimation, coregistration estimation). The spaces differ only in
**which transforms are applied at the final resampling step**.

## Shared upstream processing

These steps are computed once per run and are identical across all output
spaces:

| Step | Tool | Description |
|------|------|-------------|
| Slice timing correction | AFNI `3dTshift` | Temporal interpolation to align all slices to the middle of the TR (t = TR/2). See [STC note](#slice-timing-correction) below. |
| Head motion correction (HMC) | FSL MCFLIRT | Per-volume rigid-body (6 DOF) alignment to a reference volume (boldref). Produces `from-orig_to-boldref_desc-hmc_xfm.txt`. |
| Susceptibility distortion correction (SDC) | SDCFlows (pepolar) | B0 fieldmap estimated from AP/PA EPI pairs; stored as B-spline coefficients (`desc-coeff_fieldmap.nii.gz`). |
| Coregistration | FreeSurfer `bbregister` | Boundary-based registration of boldref to the subject's T1w anatomical. Produces `from-boldref_to-T1w_desc-coreg_xfm.txt`. |
| Spatial normalization | ANTs SyN | Non-linear warp from T1w to MNI152NLin2009cAsym template. Computed during anatomical preprocessing. |
| Confounds estimation | fMRIPrep | Framewise displacement, DVARS, CompCor components, motion parameters, cosine drift regressors. Identical across spaces. |

## Output spaces

### `func` — Native acquisition grid

Written directly by fMRIPrep 25.2.5 (the `func` output space).

| Property | Value |
|----------|-------|
| Grid | Run's original EPI grid (124 × 124 × 69) |
| Voxel size | 1.8 × 1.8 × 2.0 mm (acquisition resolution; 1.7 mm isotropic in the first participant's first two sessions; see [MRI Sequences](mri-sequences.md)) |
| Transforms applied | HMC + SDC |
| Transforms NOT applied | Coregistration, spatial normalization |
| STC | Yes (AFNI 3dTshift) — see [note below](#slice-timing-correction) |
| BIDS filename | `{prefix}_desc-preproc_bold.nii.gz` (no `space-` entity) |
| Primary use | Within-subject voxelwise analyses (GLMsingle), single-trial beta estimation |

The `func` space preserves the native acquisition grid with zero
interpolation-induced smoothing beyond the single HMC+SDC resampling
step. Each run has its own grid — cross-run alignment requires the
cached coregistration transforms.

### `T1w` — Subject anatomical space

| Property | Value |
|----------|-------|
| Grid | Acquisition resolution, on a brain-cropped grid aligned to the subject's T1w |
| Voxel size | Same as `func` |
| Transforms applied | HMC + SDC + coregistration |
| STC | Yes (AFNI 3dTshift) |
| BIDS filename | `{prefix}_space-T1w_desc-preproc_bold.nii.gz` |
| Primary use | Within-subject analyses that pool runs or sessions on one grid, or use subject anatomy (hippocampal subfields, FreeSurfer ROIs) |

All of a subject's runs share this grid, so runs pool without further
resampling.

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

### `fsnative` — Subject surface space

| Property | Value |
|----------|-------|
| Mesh | The subject's own FreeSurfer white/pial surfaces |
| Transforms applied | HMC + SDC + coregistration + cortical surface projection |
| STC | Yes (AFNI 3dTshift) |
| BIDS filename | `{prefix}_hemi-{L,R}_space-fsnative_bold.func.gii` |
| Primary use | Within-subject surface analyses at native vertex density |

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

| | func | T1w | MNI res-2 | fsnative | fsaverage6 |
|---|:---:|:---:|:---:|:---:|:---:|
| Slice timing correction | ✓ | ✓ | ✓ | ✓ | ✓ |
| Head motion correction | ✓ | ✓ | ✓ | ✓ | ✓ |
| Susceptibility distortion correction | ✓ | ✓ | ✓ | ✓ | ✓ |
| Coregistration to T1w | — | ✓ | ✓ | ✓ | ✓ |
| Spatial normalization (MNI) | — | — | ✓ | — | — |
| Surface projection | — | — | — | ✓ | ✓ |
| Interpolation steps | 1 | 1 | 1 | 1 + surface | 1 + surface |
| Cross-run alignment | via cached xfms | intrinsic | intrinsic | intrinsic | intrinsic |
| Cross-subject alignment | no | no | yes | no | yes |

## Slice timing correction

STC is applied by fMRIPrep in every output space, including `func`
(fMRIPrep runs AFNI `3dTshift` when `SliceTiming` metadata is present, before
the final resampling step, so every output space inherits it).

No BOLD without STC is provided. Producing it would require a separate
fMRIPrep run with `--ignore slicetiming`.

## Per-run transform files

Each BOLD run in `fmriprep/{sub}/{ses}/func/` includes cached
transforms that can be used to move between spaces:

| File | Description |
|------|-------------|
| `*_from-orig_to-boldref_desc-hmc_xfm.txt` | Per-volume rigid HMC transforms (ITK format, one affine per volume) |
| `*_from-boldref_to-B0map*_xfm.txt` | Rigid alignment from boldref to fieldmap space |
| `*_from-boldref_to-T1w_desc-coreg_xfm.txt` | Boundary-based coregistration (boldref → T1w) |
| `*_desc-hmc_boldref.nii.gz` | Motion-corrected reference volume |
| `*_desc-coreg_boldref.nii.gz` | Coregistered reference volume |

Fieldmap coefficients are in `fmriprep/{sub}/{ses}/fmap/`:

| File | Description |
|------|-------------|
| `*_fmapid-*_desc-coeff_fieldmap.nii.gz` | B-spline SDC coefficients |
| `*_fmapid-*_desc-epi_fieldmap.nii.gz` | EPI reference aligned with coefficients |
| `*_fmapid-*_desc-preproc_fieldmap.nii.gz` | Preprocessed fieldmap in Hz (for visualization) |

Each session has two or more fieldmaps (see
[Fieldmap-to-run mapping](#fieldmap-to-run-mapping)). The mapping from BOLD
run to fieldmap is encoded in the `from-boldref_to-B0map*` filename.

The T1w → MNI warp is computed during anatomical preprocessing and
stored in `fmriprep/{sub}/anat/`.

## Fieldmap-to-run mapping

Fieldmap IDs have the form `B0map{label}sub##ses##`. The label names the
block of runs the AP/PA pair was acquired for:

| Label | Sessions | Usually applies to |
|-------|----------|--------------------|
| `encoding` | TB and NAT sessions | encoding and math runs |
| `retrieval` | TB and NAT sessions | retrieval and resting runs |
| `first`, `second`, `third` | localizer sessions | the runs acquired after that pair |
| `encoding`, `first`, `second` | ses-01 and ses-30 | the runs acquired after that pair |
| `math`, `resting` | a few TB and NAT sessions | the runs acquired after that pair |

The usual mapping has exceptions, so read the per-run mapping from the
filenames rather than from this table.

The fieldmap ID is embedded in both the transform filename
(`from-boldref_to-B0map{id}_xfm.txt`) and the coefficient filename
(`fmapid-B0map{id}_desc-coeff_fieldmap.nii.gz`), so the mapping is
self-documenting.

---

## fMRIPrep version

The outputs described here were produced with **fMRIPrep 25.2.5** (external
FreeSurfer 8.2.0 recons). Check `dataset_description.json` in the tree you
are reading.
