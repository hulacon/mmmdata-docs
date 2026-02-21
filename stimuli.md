---
title: Stimuli
parent: Dataset Description
has_children: true
nav_order: 3
---

# Stimuli

This section describes the stimulus materials used in the MMMData experiment.
Stimuli live in the BIDS root `stimuli/` directory and are referenced by
`stim_file` paths in `_events.tsv` files.

## Overview

The experiment uses two main memory paradigms, each with distinct stimulus types:

| Paradigm | BIDS label prefix | Encoding stimuli | Retrieval stimuli |
|----------|-------------------|------------------|-------------------|
| **Trial-based (TB)** | `TB*` | Paired image + auditory word | One pairmate (image or word) as cue |
| **Naturalistic (NAT)** | `NAT*` | Movie clips | Verbal free recall (audio recorded) |

## Directory Structure

```
stimuli/
├── shared1000/                    # Visual stimuli (images + metadata)
│   ├── images/                    # 1,000 PNG files
│   ├── shared1000.csv             # Image index (mmmId, nsdId, cocoId)
│   ├── nsd_stim_info.csv          # Full NSD stimulus metadata
│   ├── coco_annotations.csv       # Per-image COCO metadata
│   ├── coco_captions.csv          # Per-caption file (5 per image)
│   ├── viz2psy_scores.csv         # Computational image features (viz2psy)
│   └── viz2psy_scores.meta.json   # Feature definitions and provenance
├── movies/                        # Movie stimuli for NAT encoding
│   ├── movie_files/               # 60 trimmed .mov files
│   ├── movie_cues/                # 60 recall cue images (.jpg)
│   ├── viz2psy_scores/            # Per-movie temporal features (viz2psy)
│   ├── MMM movies - Sheet1.csv    # Session-by-session movie schedule
│   └── short films 4 minutes.rtf  # Source notes and links
└── twp1000/                       # Auditory stimuli (spoken words)
    ├── twp1000.csv                # Word metadata (1,000-word subset)
    ├── twp_all.csv                # Full Toronto Word Pool metadata
    ├── echo/                      # Male voice 1
    ├── nova/                      # Female voice 1
    ├── onyx/                      # Male voice 2
    └── shimmer/                   # Female voice 2
```

## Computational Image Features (viz2psy)

All visual stimuli (shared1000 images, movie frames, and movie cue images) are
being processed with [viz2psy](https://github.com/hulacon/viz2psy), a toolbox
that extracts psychological and perceptual features from images and video. Output
includes ~2,900 features per image spanning memorability, emotion, scene
categorization, low-level statistics, object detection, saliency, aesthetics,
semantic embeddings (CLIP, DINOv2), spatial frequency, and captioning. Each
output CSV is accompanied by a `.meta.json` sidecar documenting feature
definitions and provenance.

## Stimulus Assignment

Image–word pairings are **randomly assigned** per subject. The specific pairings
for each subject can be reconstructed from the behavioral CSV files in
`sourcedata/sub-{id}/ses-{NN}/behavioral/`. There is no fixed pairing file;
assignment is determined at runtime by the PsychoPy experiment code.

## Open Questions

- NSD image licensing terms for data sharing
- Movie licensing/provenance details for data sharing
