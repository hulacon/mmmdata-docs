---
title: Behavioral & Physiological Data
parent: Data Acquisition
grand_parent: Dataset Description
nav_order: 2
---

# Behavioral & Physiological Data

## Behavioral Data

Behavioral response data are stored in `beh/` directories within each session.
These files capture task performance outside the BOLD time series (e.g.,
recognition judgments, timeline ordering).

### File format

Each behavioral file consists of a paired TSV and JSON sidecar:

```
sub-03/ses-04/beh/
├── sub-03_ses-04_task-TB2AFC_run-01_beh.tsv
└── sub-03_ses-04_task-TB2AFC_run-01_beh.json
```

### Tasks with behavioral files

| Task | Sessions | Description |
|------|----------|-------------|
| `TB2AFC` | ses-04 through ses-18 | Two-alternative forced choice recognition test |
| `FIN2AFC` | ses-30 | Final session 2AFC recognition |
| `FINtimeline` | ses-30 | Temporal ordering judgment |

### Key columns (TB2AFC)

`onset`, `duration`, `trial_type`, `modality`, `word`, `image1`, `image2`,
`correct_resp`, `resp`, `resp_RT`, `trial_accuracy`, `enCon` (encoding
condition), `reCon` (retrieval condition), `cueId`, `pairId`.

## Physiological Data

Physiological recordings are stored alongside functional data in `func/`
directories, using the BIDS `_physio.tsv.gz` + `_physio.json` format.

### Scanner physiological monitoring

Cardiac, pulse oximetry, and respiratory signals recorded by the Siemens PMU
system during scanning. Present primarily in trial-based sessions.

```
sub-03/ses-04/func/
├── sub-03_ses-04_task-TBencoding_run-01_recording-cardiac_physio.tsv.gz
├── sub-03_ses-04_task-TBencoding_run-01_recording-cardiac_physio.json
├── sub-03_ses-04_task-TBencoding_run-01_recording-pulse_physio.tsv.gz
├── sub-03_ses-04_task-TBencoding_run-01_recording-pulse_physio.json
├── sub-03_ses-04_task-TBencoding_run-01_recording-respiratory_physio.tsv.gz
└── sub-03_ses-04_task-TBencoding_run-01_recording-respiratory_physio.json
```

| Recording | Column | Sampling (Hz) | Source |
|-----------|--------|---------------|--------|
| `recording-cardiac` | `cardiac` | 1000 | Siemens PMU |
| `recording-pulse` | `pulse` | 1000 | Siemens PMU |
| `recording-respiratory` | `respiratory` | 1000 | Siemens PMU |

Scanner physio is most consistently available in trial-based sessions (ses-04
through ses-18). Availability varies by session and subject depending on
equipment setup and signal quality.

## Eye-tracking

Eye position and pupil size recorded with an SR Research EyeLink system during
naturalistic movie-watching and free recall sessions.

```
sub-03/ses-19/func/
├── sub-03_ses-19_task-NATencoding_run-01_recording-eye_physio.tsv.gz
├── sub-03_ses-19_task-NATencoding_run-01_recording-eye_physio.json
├── sub-03_ses-19_task-NATretrieval_recording-eye_physio.tsv.gz
└── sub-03_ses-19_task-NATretrieval_recording-eye_physio.json
```

| Recording | Columns | Sampling (Hz) | Source |
|-----------|---------|---------------|--------|
| `recording-eye` | `eye1_x_coordinate`, `eye1_y_coordinate`, `eye1_pupil_size` | 1000 | SR Research EyeLink |

Eye-tracking data is trimmed to the scan window defined by scanner triggers
(input=255). The `StartTime` in the JSON sidecar corresponds to the first
BOLD volume onset.

Eye-tracking is primarily available in naturalistic sessions (ses-19 through
ses-28). Raw EDF source files are preserved in
`sourcedata/sub-XX/ses-YY/eyetracking/`.

## Audio recordings

Verbal free recall recordings (WAV format) are stored in sourcedata and are
not part of the BIDS raw dataset:

```
sourcedata/sub-03/ses-19/audio/
├── recording_mic_2025-04-01_13h37.55.955.wav
├── recording_mic_2025-04-01_13h39.37.413.wav
└── ...
```

Audio recordings are collected during naturalistic free recall runs (ses-19
through ses-28) and the final free recall session (ses-29).
