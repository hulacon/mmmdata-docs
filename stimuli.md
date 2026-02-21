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
│   ├── nsd_stim_info.csv          # NSD stimulus metadata
│   ├── coco_annotations.csv       # Per-image COCO metadata
│   ├── coco_captions.csv          # Per-caption file (5 per image)
│   └── images/                    # 1,000 PNG files
├── movies/                        # Movie stimuli for NAT encoding
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

## Stimulus Assignment

Image–word pairings are **randomly assigned** per subject. The specific pairings
for each subject can be reconstructed from the behavioral CSV files in
`sourcedata/sub-{id}/ses-{NN}/behavioral/`. There is no fixed pairing file;
assignment is determined at runtime by the PsychoPy experiment code.

## Open Questions

- NSD image licensing terms for data sharing
- Movie licensing/provenance details for data sharing
