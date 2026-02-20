---
title: Study Overview
parent: Dataset Description
nav_order: 1
---

# Study Overview

The Multi-Modal Memory Dataset (MMMData) is a dense (many observations for a
smaller number of participants) fMRI study being conducted at the University of
Oregon. The goal is to better understand how the brain remembers sights and
sounds and how doing so in traditional, experimental settings (unrelated,
discrete series of trials) relates to doing so in more naturalistic settings
(i.e., watching and recalling a video).

- **Design**: Dense within-subject (many sessions per participant, few participants)
- **Target sample**: 6 usable subjects (sub-03 through sub-09, excluding pilots)
- **Sessions per subject**: Up to 30 scanning sessions
- **Scanner**: Siemens MAGNETOM Prisma 3T, University of Oregon
- **BIDS version**: 1.9.0

## Participants

| Subject | Status |
|---------|--------|
| sub-01, sub-02 | Pilot. Raw DICOMs preserved in `sourcedata/dicoms/` only. Not BIDSified. |
| sub-03, sub-04, sub-05 | Active. Fully BIDSified with 30 sessions each. |
| sub-06 through sub-09 | Planned. Will follow the same session schedule. |

Demographics (age, sex) will be provided later. The current `participants.tsv` is
placeholder scaffolding and does not reflect the actual subjects.
