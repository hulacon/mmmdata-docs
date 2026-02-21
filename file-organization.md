---
title: BIDS Structure
parent: Data Organization
grand_parent: Dataset Description
nav_order: 1
---

# File Organization

## BIDS Root

```
/projects/hulacon/shared/mmmdata/
├── dataset_description.json
├── participants.tsv              # Currently placeholder — needs updating
├── participants.json
├── README                        # Currently empty — needs content
├── CHANGES
├── licenses/
├── sub-03/
│   ├── ses-01/
│   │   ├── anat/                 # T1w (acq-MPR), T2w (acq-SPC, acq-oblcor)
│   │   ├── dwi/                  # dir-AP, dir-PA, dir-LR, dir-RL
│   │   └── func/                 # task-resting
│   ├── ses-02/
│   │   ├── fmap/                 # dir-AP, dir-PA spin-echo EPI pairs
│   │   └── func/                 # localizer tasks
│   ├── ses-04/ through ses-28/
│   │   ├── fmap/
│   │   └── func/                 # TB or NAT memory tasks
│   ├── ses-29/
│   │   └── beh/                  # Behavioral data (awaiting BIDSification)
│   └── ses-30/
│       ├── anat/                 # Final T1w
│       ├── fmap/
│       └── func/                 # Final memory + makeup localizers
├── sub-04/
├── sub-05/
├── derivatives/
├── sourcedata/
├── inventory/
├── code/
├── phenotype/                    # Currently empty
└── stimuli/                      # Shared stimulus files (see stimuli.md)
    ├── shared1000/               # 1,000 NSD shared images (PNG) + metadata CSVs
    │   ├── nsd_stim_info.csv     # NSD stimulus metadata (nsdId links to filenames)
    │   ├── coco_annotations.csv  # COCO captions + object categories per image
    │   ├── coco_captions.csv     # All 5,000 COCO captions (5 per image)
    │   └── images/               # The actual PNG files
    ├── movies/                   # Movie stimuli for NAT encoding (~40 short films)
    └── twp1000/                  # 1,000 Toronto Word Pool words × 4 voices (MP3)
```

## File Naming Conventions

```
sub-{id}_ses-{id}[_acq-{label}][_task-{label}][_dir-{label}][_run-{id}]_{suffix}.{ext}
```

All NIfTI files are gzipped (`.nii.gz`). Every NIfTI has a paired JSON sidecar.

## Expected Files per Datatype

| Datatype | Suffixes | Sidecars | Notes |
|----------|----------|----------|-------|
| `anat/` | `T1w.nii.gz`, `T2w.nii.gz` | `.json` | acq-MPR for T1w; acq-SPC and acq-oblcor for T2w |
| `func/` | `bold.nii.gz` | `.json`, `_events.tsv` (missing) | One per task/run |
| `dwi/` | `dwi.nii.gz` | `.json`, `.bval`, `.bvec` | Four phase-encoding directions in ses-01/ses-28 |
| `fmap/` | `epi.nii.gz` | `.json` | AP/PA pairs, typically 2 runs per session |
| `beh/` | TBD | TBD | ses-29 only, awaiting BIDSification |
