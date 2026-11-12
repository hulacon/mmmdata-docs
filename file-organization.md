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
├── participants.tsv              # Demographics, one row per participant
├── participants.json
├── README
├── CHANGES
├── licenses/
├── sub-##/
│   ├── ses-01/
│   │   ├── anat/                 # T1w (acq-MPR), T2w (acq-SPC, acq-oblcor)
│   │   ├── dwi/                  # dir-AP, dir-PA, dir-LR, dir-RL
│   │   └── func/                 # task-INITresting
│   ├── ses-02/
│   │   ├── fmap/                 # dir-AP, dir-PA spin-echo EPI pairs
│   │   └── func/                 # localizer tasks
│   ├── ses-04/ through ses-28/
│   │   ├── fmap/
│   │   ├── beh/                  # Behavioral files (TB2AFC; NATretrieval recall transcripts)
│   │   └── func/                 # TB or NAT memory tasks + events + physio
│   ├── ses-28/                   # also anat/ (T1w) and dwi/
│   ├── ses-29/
│   │   └── beh/                  # Out-of-scanner free recall (FINrecall); no imaging
│   └── ses-30/
│       ├── anat/                 # Final T1w
│       ├── fmap/
│       ├── beh/                  # FIN2AFC, FINtimeline
│       └── func/                 # Final memory + makeup localizers
├── derivatives/
├── inventory/
├── code/
├── phenotype/                    # Questionnaire & debriefing data (VVIQ, final debriefing)
├── expectations/                 # dataset.toml — declared expectations (see data-organization.md)
└── stimuli/                      # Shared stimulus files (see stimuli.md)
    ├── shared1000/               # 1,000 NSD shared images (PNG) + metadata CSVs
    │   ├── nsd_stim_info.csv     # NSD stimulus metadata (nsdId links to filenames)
    │   ├── coco_annotations.csv  # COCO captions + object categories per image
    │   ├── coco_captions.csv     # All COCO captions (5 per image, 6 for a few)
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
| `func/` | `_physio.tsv.gz` | `_physio.json` | `recording-trigger`, `recording-pulse`, `recording-respiratory`, or `recording-eye` |
| `dwi/` | `dwi.nii.gz` | `.json`, `.bval`, `.bvec` | ses-01: four phase-encoding directions (AP, PA, LR, RL); ses-28: AP and PA, sometimes all four |
| `fmap/` | `epi.nii.gz` | `.json` (with `B0FieldIdentifier`) | AP/PA pairs, typically 2 runs per session |
| `beh/` | `_beh.tsv` | `_beh.json` | Behavioral response files (TB2AFC in ses-04–18, FIN2AFC/FINtimeline in ses-30) and NATretrieval recall transcripts (one row per spoken word) |

## Task labels

Functional task labels carry a phase prefix. The full set in use:

| Prefix | Labels |
|---|---|
| Baseline | `INITresting` |
| Trial-based | `TBencoding`, `TBretrieval`, `TBmath`, `TBresting`, `TB2AFC` (beh) |
| Naturalistic | `NATencoding`, `NATretrieval`, `NATmath`, `NATresting` |
| Final | `FINrecall` (beh), `FINretrieval`, `FINresting`, `FIN2AFC` (beh), `FINtimeline` (beh) |
| Localizer | `prf`, `floc`, `tone`, `auditory`, `motor`, `fixation` |

`TB2AFC`, `FINrecall`, `FIN2AFC` and `FINtimeline` appear only as `beh/` files — they have
no BOLD runs. `NATretrieval` has both: BOLD runs in `func/` and a word-level
recall transcript per session in `beh/`.

---

*Which sessions exist for each subject is recorded in the catalog (see
[Data Organization](data-organization.md)).*
