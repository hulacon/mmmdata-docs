---
title: Output Spaces & Preprocessing Steps
parent: Data Organization
grand_parent: Dataset Description
nav_order: 5
---

# Output Spaces & Preprocessing Steps

> **All three spaces below exist.** The re-preprocessing campaign that
> finished **2026-08-26** produced fMRIPrep **25.2.5** output in
> `derivatives/fmriprep/` with `func` (native acquisition grid),
> `MNI152NLin2009cAsym res-2` and `fsaverage6` for every run it covered.
> fMRIPrep 25.2.5 writes the `func` space directly, so the earlier post-hoc
> resampling path is retired. The former `fmriprep_nordic/` tree was deleted
> on the same date (see [Derivatives](derivatives.md#nordic-arm-retired)).
> Query the catalog for which runs are covered.

This page describes three output spaces. Each
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
| Voxel size | 1.7 mm isotropic (acquisition resolution; see [MRI Sequences](mri-sequences.md)) |
| Transforms applied | HMC + SDC |
| Transforms NOT applied | Coregistration, spatial normalization |
| STC | Yes (AFNI 3dTshift) — see [note below](#slice-timing-correction) |
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
| Slice timing correction | ✓ | ✓ | ✓ |
| Head motion correction | ✓ | ✓ | ✓ |
| Susceptibility distortion correction | ✓ | ✓ | ✓ |
| Coregistration to T1w | — | ✓ | ✓ |
| Spatial normalization (MNI) | — | ✓ | — |
| Surface projection (fsaverage6) | — | — | ✓ |
| Interpolation steps | 1 | 1 | 1 + surface |
| Cross-run alignment | via cached xfms | intrinsic | intrinsic |
| Cross-subject alignment | no | yes | yes |
| **Exists on disk** | **yes** | **yes** | **yes** |

## Slice timing correction

STC is applied by fMRIPrep in all three output spaces, including `func`
(fMRIPrep runs AFNI `3dTshift` when `SliceTiming` metadata is present, before
the final resampling step, so every output space inherits it).

An earlier version of this page described a `func` space produced by a
post-hoc resampling script with STC intentionally skipped, on the reasoning
that GLMsingle's per-voxel HRF fitting absorbs slice-timing delays and that
STC's temporal interpolation can reduce tSNR for single-trial estimation. That
path is retired: fMRIPrep 25.2.5 writes `func` directly, and no non-STC'd
BOLD exists on disk. Producing one would require a separate run with
`--ignore slicetiming`.

**Open question:** whether STC helps or hurts GLMsingle voxel reliability on
this dataset has not been tested empirically. That comparison remains open; it
is not a decision.

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

Two fieldmaps per session: one for encoding runs (`B0mapencodingXXX`),
one for retrieval/resting runs (`B0mapretrievalXXX`). The mapping from
BOLD run to fieldmap is encoded in the `from-boldref_to-B0map*` filename.

The T1w → MNI warp is computed during anatomical preprocessing and
stored in `fmriprep/{sub}/anat/`.

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

---

## fMRIPrep version

The outputs described here were produced with **fMRIPrep 25.2.5** (external
FreeSurfer 8.2.0 recons) in the re-preprocessing campaign that finished
2026-08-26. No 24.1.1 output remains anywhere in the dataset. Check
`dataset_description.json` in the tree you are reading.

*Space inventory on this page verified against the filesystem on 2026-08-20;
space, STC, voxel-size and version claims updated 2026-09-05 from the
reprocessing-campaign record.*
