---
title: BIDS Structure
parent: Data Organization
grand_parent: Dataset Description
nav_order: 1
---

# File Organization

## BIDS Root

```
/gpfs/projects/hulacon/shared/mmmdata/
├── dataset_description.json
├── participants.tsv              # Full demographics for sub-03/04/05
├── participants.json
├── README                        # Empty — still needs dataset description
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
│   │   ├── beh/                  # Behavioral response files (e.g., TB2AFC)
│   │   └── func/                 # TB or NAT memory tasks + events + physio
│   ├── ses-29/                   # Out-of-scanner behavioral session (BIDSification in progress)
│   └── ses-30/
│       ├── anat/                 # Final T1w
│       ├── fmap/
│       └── func/                 # Final memory + makeup localizers
├── sub-04/
├── sub-05/
├── derivatives/
├── inventory/
├── code/
├── phenotype/                    # Questionnaire & debriefing data (VVIQ, final debriefing)
├── expectations/                 # dataset.toml — declared expectations (see data-organization.md)
└── stimuli/                      # Shared stimulus files (see stimuli.md)
    ├── shared1000/               # 1,000 NSD shared images (PNG) + metadata CSVs
    │   ├── nsd_stim_info.csv     # NSD stimulus metadata (nsdId links to filenames)
    │   ├── coco_annotations.csv  # COCO captions + object categories per image
    │   ├── coco_captions.csv     # All 5,000 COCO captions (5 per image)
    │   └── images/               # The actual PNG files
    ├── movies/                   # Movie stimuli for NAT encoding (60 short films)
    ├── twp1000/                  # 1,000 Toronto Word Pool words × 4 voices (MP3)
    └── stimulus_registry/        # Canonical stimulus_id tables (see stimuli_features.md)
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
| `func/` | `bold.nii.gz`, `sbref.nii.gz` | `.json`, `_events.tsv`, `_events.json` | One set per task/run |
| `func/` | `_physio.tsv.gz` | `_physio.json` | `recording-cardiac`, `recording-pulse`, `recording-respiratory`, or `recording-eye` |
| `dwi/` | `dwi.nii.gz` | `.json`, `.bval`, `.bvec` | Four phase-encoding directions in ses-01/ses-28 |
| `fmap/` | `epi.nii.gz` | `.json` (with `B0FieldIdentifier`) | AP/PA pairs, typically 2 runs per session |
| `beh/` | `_beh.tsv` | `_beh.json` | Behavioral response files (e.g., TB2AFC in ses-04–18) |

## Task labels

Functional task labels carry a phase prefix. The full set in use:

| Prefix | Labels |
|---|---|
| Baseline | `INITresting` |
| Trial-based | `TBencoding`, `TBretrieval`, `TBmath`, `TBresting`, `TB2AFC` (beh) |
| Naturalistic | `NATencoding`, `NATretrieval`, `NATmath`, `NATresting` |
| Final | `FINretrieval`, `FINresting`, `FIN2AFC` (beh), `FINtimeline` (beh) |
| Localizer | `prf`, `floc`, `tone`, `auditory`, `motor`, `fixation` |

`TB2AFC`, `FIN2AFC` and `FINtimeline` appear only as `beh/` files — they have
no BOLD runs.

---

*Structure on this page verified against the filesystem on 2026-08-20.
Per-subject session coverage is deliberately not asserted here — query the
catalog (see [Data Organization](data-organization.md)).*
