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

| Subject | Age | Sex | Handedness | Vision | Status |
|---------|-----|-----|------------|--------|--------|
| sub-01, sub-02 | — | — | — | — | Pilot. DICOMs in `sourcedata/dicoms/` only. Not BIDSified. |
| sub-03 | 20 | M | right | normal | Active. Fully BIDSified (ses-01–28, ses-30). |
| sub-04 | 28 | M | right | normal | Active. Fully BIDSified (ses-01–28, ses-30). |
| sub-05 | 22 | F | right | corrected | Active. Fully BIDSified (ses-01–28, ses-30). |
| sub-06 through sub-09 | — | — | — | — | Planned. Will follow the same session schedule. |

Full demographics (including education, race, sleep, medication history) are
recorded in `participants.tsv`. Questionnaire data (VVIQ, final debriefing)
are in `phenotype/`.
