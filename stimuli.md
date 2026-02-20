---
title: Stimuli
parent: Dataset Description
nav_order: 7
---

# Stimuli

This document describes the stimulus materials used in the MMMData experiment.
Stimuli live in the BIDS root `stimuli/` directory and are referenced by
`stim_file` paths in `_events.tsv` files.

## Overview

The experiment uses two main memory paradigms, each with distinct stimulus types:

| Paradigm | BIDS label prefix | Encoding stimuli | Retrieval stimuli |
|----------|-------------------|------------------|-------------------|
| **Trial-based (TB)** | `TB*` | Paired image + auditory word | One pairmate (image or word) as cue |
| **Naturalistic (NAT)** | `NAT*` | Movie clips | Verbal free recall (audio recorded) |

During TB encoding, a single word is played through headphones while an image is
presented on screen. Image–word pairings are randomly assigned per subject and
can be reconstructed from the behavioral/events data. During TB cued recall, the
participant receives one pairmate (either the auditory word or the visual image)
and must recall the other.

## Directory Structure

```
stimuli/
├── shared1000/                    # Visual stimuli (images + metadata)
│   ├── nsd_stim_info.csv          # NSD stimulus metadata (73,000 images; shared1000=True for this subset)
│   ├── coco_annotations.csv       # Per-image COCO metadata: captions, object categories, supercategories
│   ├── coco_captions.csv          # Per-caption file (5 captions per image, 5,000 rows)
│   └── images/
│       ├── shared0001_nsd02951.png
│       ├── shared0002_nsd02991.png
│       └── ...                    # 1,000 images total
├── movies/                        # Movie stimuli for NAT encoding
│   ├── MMM movies - Sheet1.csv    # Session-by-session movie schedule + recall conditions
│   └── short films 4 minutes.rtf  # Source notes and Google Drive links for movie files
└── twp1000/                       # Auditory stimuli (spoken words)
    ├── twp1000.csv                # Word metadata (subset used in this study)
    ├── twp_all.csv                # Full Toronto Word Pool metadata
    ├── echo/                      # Male voice 1
    │   ├── abide_echo.mp3
    │   └── ...
    ├── nova/                      # Female voice 1
    │   ├── abide_nova.mp3
    │   └── ...
    ├── onyx/                      # Male voice 2
    │   ├── abide_onyx.mp3
    │   └── ...
    └── shimmer/                   # Female voice 2
        ├── abide_shimmer.mp3
        └── ...
```

## Visual Stimuli: shared1000

- **Source**: Natural Scenes Dataset (NSD) shared images
  (see [NSD Experiments documentation](https://cvnlab.slite.page/p/NKalgWd__F/Experiments))
- **Count**: 1,000 images (in `images/` subdirectory)
- **Format**: PNG
- **Naming**: `shared{NNNN}_nsd{NNNNN}.png`
  — `shared` index (1-based, zero-padded to 4 digits) + original NSD image ID
- **Selection**: These are the 1,000 images designated as "shared" in the NSD
  experiment (shown to all NSD participants). Selection criteria and image
  properties are documented by the NSD group.
- **Licensing**: TBD — follows NSD data sharing terms

### Image Metadata

`nsd_stim_info.csv` contains metadata for the full NSD stimulus set (73,000
images). Rows where `shared1000=True` correspond to the images used in this
study.

| Column | Description |
|--------|-------------|
| (index) | 0-based row index |
| `cocoId` | MS-COCO image ID |
| `cocoSplit` | MS-COCO dataset split (e.g., `val2017`) |
| `cropBox` | Crop coordinates used to extract the stimulus from the original image |
| `loss` | NSD loss metric for the image |
| `nsdId` | NSD image ID — matches the `_nsd{NNNNN}` portion of the PNG filename |
| `flagged` | Whether the image was flagged in NSD |
| `BOLD5000` | Whether the image appears in the BOLD5000 dataset |
| `shared1000` | **True** for images in this study's stimulus set |

### COCO Annotations

All 1,000 shared images originate from the MS-COCO `train2017` split. COCO
annotations (captions and object instances) have been extracted for these images
and stored in two CSV files.

**`coco_annotations.csv`** — one row per image (1,000 rows):

| Column | Description |
|--------|-------------|
| `nsdId` | NSD image ID (links to filenames and `nsd_stim_info.csv`) |
| `cocoId` | MS-COCO image ID |
| `caption_1` through `caption_5` | Five human-written captions from COCO |
| `object_categories` | Semicolon-separated list of COCO object categories detected in the image |
| `object_counts` | Category:count pairs (e.g., `person:3; dog:1`) |
| `supercategories` | Semicolon-separated COCO supercategories (e.g., `animal; person`) |
| `n_object_instances` | Total number of annotated object instances |
| `n_unique_categories` | Number of distinct object categories |

**`coco_captions.csv`** — one row per caption (5,000+ rows):

| Column | Description |
|--------|-------------|
| `nsdId` | NSD image ID |
| `cocoId` | MS-COCO image ID |
| `caption_index` | Caption number (1–5) |
| `caption` | The caption text |

These files were generated by `scripts/extract_coco_metadata.py` from the
official COCO `annotations_trainval2017` release. The 80 COCO object categories
span supercategories including person, vehicle, outdoor, animal, accessory,
sports, kitchen, food, furniture, electronic, appliance, and indoor.

## Auditory Stimuli: twp1000 (Toronto Word Pool)

- **Source**: Toronto Word Pool, a standard psycholinguistic word set used
  across many memory experiments
- **Count**: 1,000 words × 4 speakers = 4,000 audio files
- **Format**: MP3
- **Naming**: `{word}_{voice}.mp3` (e.g., `cabin_shimmer.mp3`)
- **Generation**: Text-to-speech via OpenAI API (GPT-4 Turbo model),
  generated November 2023

### Speakers

| Voice ID | Gender | OpenAI TTS voice |
|----------|--------|------------------|
| `echo`   | Male   | Echo             |
| `nova`   | Female | Nova             |
| `onyx`   | Male   | Onyx             |
| `shimmer`| Female | Shimmer          |

### Word Metadata

Two CSV files provide psycholinguistic properties for each word:

- **`twp1000.csv`** — the 1,000-word subset used in this study
- **`twp_all.csv`** — the full Toronto Word Pool

| Column | Description |
|--------|-------------|
| `itmno` | Item number (1-based index) |
| `word` | The word itself |
| `imagery` | Imagery rating |
| `concreteness` | Concreteness rating |
| `letters` | Number of letters |
| `frequency` | Word frequency |
| `foa` | TBD — awaiting clarification |
| `soa` | TBD — awaiting clarification |
| `onr` | TBD — awaiting clarification |
| `dictcode` | TBD — awaiting clarification |
| `noun` | Percentage classified as noun |
| `canadian` | TBD — awaiting clarification |
| `tmpno` | TBD — present in `twp1000.csv` only |

## Stimulus Assignment

Image–word pairings are **randomly assigned** per subject. The specific pairings
for each subject can be reconstructed from the behavioral CSV files in
`sourcedata/sub-{id}/ses-{NN}/behavioral/`. There is no fixed pairing file;
assignment is determined at runtime by the PsychoPy experiment code.

## Movie Stimuli (NAT Paradigm)

Short films (~4 minutes each) shown during naturalistic encoding sessions
(ses-19 through ses-28). The movie schedule is defined in
`movies/MMM movies - Sheet1.csv`.

- **Count**: ~40 unique titles across 10 sessions
- **Duration**: ~4 minutes each
- **Styles**: Animated, live-action, and stop-motion; with or without speech
- **Source**: Mix of short films and clips from longer films (sourced via a
  "memory search project" with collaborators Jordan & Kahlyn). Movie files are
  stored on Google Drive (links in `short films 4 minutes.rtf`).
- **Movie files**: 60 `.mov` files in `movies/movie_files/`, named
  `{Title}_trimmed_normalized_filtered.mov`
- **Movie cues**: 60 `.jpg` recall cue images in `movies/movie_cues/`, named
  `{Title}_cue.jpg`

### Recall Conditions

Each movie in a session is assigned one of three recall conditions:

| Condition | Label | Description |
|-----------|-------|-------------|
| 1 | Same-session recall | Viewed and recalled within the same session |
| 2 | Next-session recall | Viewed this session, recalled in the following session |
| 3 | Multiple-repeat | Recurring film shown across many/all sessions |

Two films recur across all 10 sessions as condition 3: **"From Dad To Son"**
(animated, no speech) and **"The Bench"** (live-action, speech).

### Session Structure

Each session contains 6–8 movies:
- ~4 **condition 1** movies (same-session recall)
- ~2 **condition 2** movies (recalled next session)
- 2 **condition 3** movies (recurring)
- Sessions 2–10 also include 2 **carry-over** movies from the prior session
  (positions 7–8 at recall) that are recalled but not re-viewed

### Movie Schedule Metadata

`MMM movies - Sheet1.csv` columns:

| Column | Description |
|--------|-------------|
| `Session` | NAT session number (1–10, mapping to ses-19 through ses-28) |
| `Condition` | Recall condition: 1 = same-session, 2 = next-session, 3 = multi-repeat |
| `Movie name` | Title of the film |
| `Movie style` | Style description (e.g., "Animated, no speech", "Live-action, speech") |
| `Movie duration` | Duration (currently empty — all ~4 minutes) |
| `Cue position at Recall` | Order position (1–8) of this movie's recall cue; N/A for condition-2 movies |

Blank rows in the CSV separate sessions. Rows without a condition value are
carry-over movies from the prior session.

## Open Questions

- Full column definitions for the TWP metadata CSVs (`foa`, `soa`, `onr`,
  `dictcode`, `canadian`, `tmpno`)
- NSD image licensing terms for data sharing
- Movie licensing/provenance details for data sharing
