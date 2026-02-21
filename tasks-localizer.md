---
title: Localizer Tasks
parent: Experimental Design
grand_parent: Dataset Description
nav_order: 2
---

# Functional Localizer Tasks

Collected during ses-02, ses-03, and sometimes ses-30 (makeup runs).

| BIDS task label | Description |
|-----------------|-------------|
| `task-prf` | Population receptive field mapping (typically 3 runs) |
| `task-floc` | Functional localizer (3-6 runs) |
| `task-auditory` | Auditory stimulation |
| `task-tone` | Tonotopy (1-2 runs) |
| `task-motor` | Motor task (2 runs, ses-30) |
| `task-fixation` | Fixation baseline (ses-30) |

These labels are expected to remain unchanged.

## PRF
- 3 runs for each session
- Order to run (93=multibar, 94=wedgeringmash):
  - Run 1: 93, 94, 93
  - Run 2: 94, 93, 94
- Code adapted from http://kendrickkay.net/analyzePRF/
- Two run types were used: multibar and wedgering. The first, termed 'multibar', involves bars sweeping in multiple directions (same as RETBAR in the Human Connectome Project 7T Retinotopy Dataset). The second, termed 'wedgering', involves a combination of rotating wedges and expanding and contracting rings.
- A total of 6 runs (3 multibar and 3 wedgering) were collected across two functional localizer sessions. Each run lasted 300 s.
- The acquisition structure was [MWM] for the first localizer session, and [WMW] for the second localizer session. M indicates a multibar run and W indicates a wedgering run.
- Stimuli filled a circular region with diameter 10.6°. Throughout the stimulus presentation, a small semi-transparent dot (0.2° × 0.2°) was present at the center of the stimuli.
- Participants were instructed to maintain fixation on a central dot (switched randomly to one of three colors: black, white, or red, every 1–5 s) and press a button whenever the color of the dot changes.

## fLoc
- 3 runs for each session

## Motor Localizer
- 1 run for the 2nd localizer session and during the final cued recall session

## Auditory Category Localizer
- 1 run for the 1st localizer session and during the final cued recall session

## Tonotopy Mapping
- 1 run for each session
- Order to run (low to high or high to low):
  - Run 1: low to high
  - Run 2: high to low
